---
title : "Automatic Reminders with EventBridge and SES"
date : "2026-07-07"
weight : 10
chapter : false
pre : " <b> 5.10 </b> "
---

#### Content
- [How it works](#how-it-works)
- [Step 1: Verify an email in SES](#step-1-verify-an-email-in-ses)
- [Step 2: Create the SQS Dead Letter Queue](#step-2-create-the-sqs-dead-letter-queue)
- [Step 3: Create the Follow-up Checker Lambda](#step-3-create-the-follow-up-checker-lambda)
- [Step 4: Test manually](#step-4-test-manually)
- [Step 5: Create the EventBridge Schedule](#step-5-create-the-eventbridge-schedule)

#### How it works

```
EventBridge Scheduler (9:00 AM cron)
        ↓
Follow-up Checker Lambda
        ↓ Query the FollowUpIndex GSI
DynamoDB (followUpStatus = PENDING AND followUpDate <= today)
        ↓
Amazon SES → reminder email
        ↓
Set followUpStatus = NOTIFIED (prevents duplicates)
```

If the Lambda fails: it retries twice → still failing → the message lands in the **SQS DLQ**.

#### Step 1: Verify an email in SES

{{% notice info %}}
SES starts in **sandbox mode**: it can only send to verified addresses. For this workshop, verifying the team's emails is enough — production access is not required.
{{% /notice %}}

1. Go to **Amazon SES** (confirm the region is `ap-southeast-1`)
2. Left menu → **Identities** → **Create identity**
3. **Identity type**: choose **Email address**
4. Enter the address you will use as the **sender** → **Create identity**
5. Open your inbox and click the link in "Email Address Verification Request"
6. Back in the Console, the status turns **Verified**
7. Repeat for the address that will **receive** reminders (the app login email)

![SES verified](/images/5-Workshop/5.10-Reminder-EventBridge-SES/ses-verified.png)

#### Step 2: Create the SQS Dead Letter Queue

1. Go to **Amazon SQS** → **Create queue**
2. **Type**: **Standard**
3. **Name**: `followup-dlq`
4. Keep all other defaults → **Create queue**

#### Step 3: Create the Follow-up Checker Lambda

Create the function `lambda-followup-checker` (Node.js 20.x, arm64):

```javascript
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient, QueryCommand, UpdateCommand } from "@aws-sdk/lib-dynamodb";
import { SESClient, SendEmailCommand } from "@aws-sdk/client-ses";

const ddb = DynamoDBDocumentClient.from(new DynamoDBClient({}));
const ses = new SESClient({});
const TABLE = process.env.TABLE_NAME;
const FROM = process.env.SES_FROM;

export const handler = async () => {
  const today = new Date().toISOString().slice(0, 10);

  // Query the GSI instead of scanning: read only due PENDING jobs
  const out = await ddb.send(new QueryCommand({
    TableName: TABLE,
    IndexName: "FollowUpIndex",
    KeyConditionExpression: "followUpStatus = :p AND followUpDate <= :today",
    ExpressionAttributeValues: { ":p": "PENDING", ":today": today },
  }));

  const jobs = out.Items || [];
  console.log(`Found ${jobs.length} jobs due for follow-up (<= ${today})`);

  let sent = 0;
  for (const job of jobs) {
    if (!job.userEmail) continue;
    try {
      await ses.send(new SendEmailCommand({
        Source: FROM,
        Destination: { ToAddresses: [job.userEmail] },
        Message: {
          Subject: {
            Data: `[Job Tracker] Follow-up due: ${job.position} @ ${job.company}`,
            Charset: "UTF-8",
          },
          Body: {
            Text: {
              Charset: "UTF-8",
              Data: `Hello,

Today (${today}) is the follow-up date for your application:

  • Position: ${job.position}
  • Company:  ${job.company}
  • Status:   ${job.status}
  • Applied:  ${job.appliedDate}

Don't forget to check in with the recruiter!

— Job Application Tracker`,
            },
          },
        },
      }));
      // Mark as notified so the next run does not send a duplicate
      await ddb.send(new UpdateCommand({
        TableName: TABLE,
        Key: { userId: job.userId, jobId: job.jobId },
        UpdateExpression: "SET followUpStatus = :n, updatedAt = :t",
        ExpressionAttributeValues: { ":n": "NOTIFIED", ":t": new Date().toISOString() },
      }));
      sent++;
    } catch (err) {
      // Throwing makes the async invocation fail → message goes to the DLQ
      console.error(`Failed to send mail for job ${job.jobId}:`, err);
      throw err;
    }
  }

  return { checked: jobs.length, sent };
};
```

**Configuration:**

1. **Environment variables**: `TABLE_NAME=JobTrackerJobs`, `SES_FROM=<verified sender email>`
2. **Configuration → General configuration → Edit**: change **Timeout** from 3 seconds to **30 seconds**
3. **IAM inline policy** (replace `<ACCOUNT_ID>`):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["dynamodb:Query", "dynamodb:UpdateItem"],
      "Resource": [
        "arn:aws:dynamodb:ap-southeast-1:<ACCOUNT_ID>:table/JobTrackerJobs",
        "arn:aws:dynamodb:ap-southeast-1:<ACCOUNT_ID>:table/JobTrackerJobs/index/FollowUpIndex"
      ]
    },
    { "Effect": "Allow", "Action": "ses:SendEmail", "Resource": "*" },
    {
      "Effect": "Allow",
      "Action": "sqs:SendMessage",
      "Resource": "arn:aws:sqs:ap-southeast-1:<ACCOUNT_ID>:followup-dlq"
    }
  ]
}
```

4. **Attach the DLQ**: **Configuration** tab → **Asynchronous invocation** → **Edit** → **Dead-letter queue service**: choose **Amazon SQS** → select `followup-dlq` → **Save**

{{% notice tip %}}
Asynchronous invocation defaults to **2 retries** before pushing a message to the DLQ. This prevents data loss from transient failures (e.g. SES throttling).
{{% /notice %}}

#### Step 4: Test manually

1. Open the app and sign in with an email **verified in SES**
2. Create a job whose **follow-up date is today**
3. Go to the Lambda → **Test** tab → **Create new event**:
   - **Event name**: `manual-test`
   - **Event JSON**: `{}`
4. Choose **Test**

**Expected result:**

```json
{ "checked": 1, "sent": 1 }
```

Open your inbox (check Spam too) → the reminder email is there.

Run **Test a second time** → the result is `{ "checked": 0, "sent": 0 }` because the job is now `NOTIFIED`. This is the **duplicate-prevention** mechanism: the cron runs daily but users receive exactly one email per follow-up date. When a user changes the follow-up date, the CRUD Lambda resets the status to `PENDING`.

![Reminder email](/images/5-Workshop/5.10-Reminder-EventBridge-SES/reminder-email.png)

#### Step 5: Create the EventBridge Schedule

1. Go to **Amazon EventBridge** → left menu **Scheduler** → **Schedules** → **Create schedule**
2. **Schedule name**: `daily-followup-check`
3. **Schedule group**: `default`
4. **Occurrence**: choose **Recurring schedule**
5. **Schedule type**: **Cron-based schedule**
   - Minutes: `0`, Hours: `9`, Day of month: `*`, Month: `*`, Day of week: `?`, Year: `*`
6. **Time zone**: choose **(UTC+07:00) Asia/Saigon**
7. **Flexible time window**: **Off**
8. **Next** → **Target detail**: choose **AWS Lambda** → **Invoke** → select `lambda-followup-checker`
9. **Next** → **Action after schedule completion**: **NONE**
10. **Permissions**: choose **Create new role for this schedule**
11. **Next** → **Create schedule**

After creation the page shows **Next 10 trigger dates** — confirming it runs at 9:00 AM every day.

![EventBridge schedule](/images/5-Workshop/5.10-Reminder-EventBridge-SES/eventbridge-schedule.png)

{{% notice tip %}}
To prove the system runs on its own: the evening before, create a job with tomorrow's `followUpDate`. At exactly 9:00 the next morning the email arrives with no manual action. Check the Lambda's CloudWatch Logs to see the invocation from the Scheduler.
{{% /notice %}}
