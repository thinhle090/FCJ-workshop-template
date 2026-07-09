---
title : "Nhắc nhở tự động với EventBridge và SES"
date : "2026-07-07"
weight : 10
chapter : false
pre : " <b> 5.10 </b> "
---

#### Nội dung
- [Cơ chế hoạt động](#cơ-chế-hoạt-động)
- [Bước 1: Verify email trong SES](#bước-1-verify-email-trong-ses)
- [Bước 2: Tạo SQS Dead Letter Queue](#bước-2-tạo-sqs-dead-letter-queue)
- [Bước 3: Tạo Lambda Follow-up Checker](#bước-3-tạo-lambda-follow-up-checker)
- [Bước 4: Kiểm thử thủ công](#bước-4-kiểm-thử-thủ-công)
- [Bước 5: Tạo EventBridge Schedule](#bước-5-tạo-eventbridge-schedule)

#### Cơ chế hoạt động

```
EventBridge Scheduler (cron 9:00 sáng)
        ↓
Lambda Follow-up Checker
        ↓ Query GSI FollowUpIndex
DynamoDB (followUpStatus = PENDING AND followUpDate <= hôm nay)
        ↓
Amazon SES → email nhắc nhở
        ↓
Cập nhật followUpStatus = NOTIFIED (chống gửi trùng)
```

Nếu Lambda lỗi: retry 2 lần → vẫn thất bại → message rơi vào **SQS DLQ**.

#### Bước 1: Verify email trong SES

{{% notice info %}}
SES mặc định ở **sandbox mode**: chỉ gửi được tới địa chỉ đã verify. Với workshop, verify email các thành viên là đủ. Không cần xin production access.
{{% /notice %}}

1. Vào **Amazon SES** (kiểm tra region là `ap-southeast-1`)
2. Menu trái chọn **Identities** → **Create identity**
3. **Identity type**: chọn **Email address**
4. Nhập email sẽ dùng làm **người gửi** → **Create identity**
5. Mở hộp thư, bấm link trong email "Email Address Verification Request"
6. Quay lại Console, trạng thái chuyển **Verified**
7. Lặp lại cho email sẽ **nhận** thư nhắc nhở (email đăng nhập app)

![SES verified](/images/5-Workshop/5.10-Reminder-EventBridge-SES/ses-verified.png)

#### Bước 2: Tạo SQS Dead Letter Queue

1. Vào **Amazon SQS** → **Create queue**
2. **Type**: **Standard**
3. **Name**: `followup-dlq`
4. Giữ mặc định các thiết lập còn lại → **Create queue**

#### Bước 3: Tạo Lambda Follow-up Checker

Tạo function `lambda-followup-checker` (Node.js 20.x, arm64):

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

  // Query GSI thay vì Scan: chỉ đọc các job PENDING đến hạn
  const out = await ddb.send(new QueryCommand({
    TableName: TABLE,
    IndexName: "FollowUpIndex",
    KeyConditionExpression: "followUpStatus = :p AND followUpDate <= :today",
    ExpressionAttributeValues: { ":p": "PENDING", ":today": today },
  }));

  const jobs = out.Items || [];
  console.log(`Tìm thấy ${jobs.length} job đến hạn follow-up (<= ${today})`);

  let sent = 0;
  for (const job of jobs) {
    if (!job.userEmail) continue;
    try {
      await ses.send(new SendEmailCommand({
        Source: FROM,
        Destination: { ToAddresses: [job.userEmail] },
        Message: {
          Subject: {
            Data: `[Job Tracker] Đến hạn follow-up: ${job.position} @ ${job.company}`,
            Charset: "UTF-8",
          },
          Body: {
            Text: {
              Charset: "UTF-8",
              Data: `Xin chào,

Hôm nay (${today}) là hạn follow-up cho đơn ứng tuyển:

  • Vị trí:  ${job.position}
  • Công ty: ${job.company}
  • Trạng thái: ${job.status}
  • Ngày nộp: ${job.appliedDate}

Đừng quên gửi email hỏi thăm nhà tuyển dụng nhé!

— Job Application Tracker`,
            },
          },
        },
      }));
      // Đánh dấu đã gửi để lần chạy sau không gửi trùng
      await ddb.send(new UpdateCommand({
        TableName: TABLE,
        Key: { userId: job.userId, jobId: job.jobId },
        UpdateExpression: "SET followUpStatus = :n, updatedAt = :t",
        ExpressionAttributeValues: { ":n": "NOTIFIED", ":t": new Date().toISOString() },
      }));
      sent++;
    } catch (err) {
      // Ném lỗi ra ngoài → invoke async thất bại → message vào DLQ
      console.error(`Gửi mail thất bại cho job ${job.jobId}:`, err);
      throw err;
    }
  }

  return { checked: jobs.length, sent };
};
```

**Cấu hình:**

1. **Environment variables**: `TABLE_NAME=JobTrackerJobs`, `SES_FROM=<email người gửi đã verify>`
2. **Configuration → General configuration → Edit**: đổi **Timeout** từ 3 giây thành **30 giây**
3. **IAM inline policy** (thay `<ACCOUNT_ID>`):

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

4. **Gắn DLQ**: tab **Configuration** → **Asynchronous invocation** → **Edit** → **Dead-letter queue service**: chọn **Amazon SQS** → chọn queue `followup-dlq` → **Save**

{{% notice tip %}}
Mặc định của asynchronous invocation: **retry 2 lần** trước khi đẩy message vào DLQ. Cơ chế này tránh mất dữ liệu vì lỗi tạm thời (ví dụ SES throttle).
{{% /notice %}}

#### Bước 4: Kiểm thử thủ công

1. Mở app, đăng nhập bằng email **đã verify trong SES**
2. Tạo một job có **Ngày follow-up = hôm nay**
3. Vào Lambda → tab **Test** → **Create new event**:
   - **Event name**: `manual-test`
   - **Event JSON**: `{}`
4. Chọn **Test**

**Kết quả mong đợi:**

```json
{ "checked": 1, "sent": 1 }
```

Mở hộp thư (kiểm tra cả Spam) → thấy email nhắc nhở.

Chạy **Test lần thứ hai** → kết quả `{ "checked": 0, "sent": 0 }` vì job đã chuyển `NOTIFIED`. Đây là cơ chế **chống gửi trùng**: cron chạy mỗi ngày nhưng người dùng chỉ nhận đúng một email cho mỗi hạn follow-up. Khi người dùng đổi ngày follow-up, Lambda CRUD tự reset trạng thái về `PENDING`.

![Reminder email](/images/5-Workshop/5.10-Reminder-EventBridge-SES/reminder-email.png)

#### Bước 5: Tạo EventBridge Schedule

1. Vào **Amazon EventBridge** → menu trái **Scheduler** → **Schedules** → **Create schedule**
2. **Schedule name**: `daily-followup-check`
3. **Schedule group**: `default`
4. **Occurrence**: chọn **Recurring schedule**
5. **Schedule type**: **Cron-based schedule**
   - Minutes: `0`, Hours: `9`, Day of month: `*`, Month: `*`, Day of week: `?`, Year: `*`
6. **Time zone**: chọn **(UTC+07:00) Asia/Saigon**
7. **Flexible time window**: **Off**
8. **Next** → **Target detail**: chọn **AWS Lambda** → **Invoke** → chọn function `lambda-followup-checker`
9. **Next** → **Action after schedule completion**: **NONE**
10. **Permissions**: chọn **Create new role for this schedule**
11. **Next** → **Create schedule**

Màn hình sau khi tạo hiển thị **Next 10 trigger dates** — xác nhận lịch chạy 9:00 sáng mỗi ngày.

![EventBridge schedule](/images/5-Workshop/5.10-Reminder-EventBridge-SES/eventbridge-schedule.png)

{{% notice tip %}}
Để có bằng chứng hệ thống chạy tự động: tối hôm trước, tạo một job có `followUpDate` là ngày mai. Sáng hôm sau đúng 9:00, email tự đến mà không ai thao tác gì. Kiểm tra CloudWatch Logs của Lambda để thấy log invoke từ Scheduler.
{{% /notice %}}
