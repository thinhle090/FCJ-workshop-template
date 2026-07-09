---
title : "Monitoring and Auditing"
date : "2026-07-07"
weight : 11
chapter : false
pre : " <b> 5.11 </b> "
---

#### Content
- [Step 1: Create an SNS Topic](#step-1-create-an-sns-topic)
- [Step 2: Create a CloudWatch Alarm](#step-2-create-a-cloudwatch-alarm)
- [Step 3: Enable CloudTrail](#step-3-enable-cloudtrail)
- [The admin role in this system](#the-admin-role-in-this-system)

#### Step 1: Create an SNS Topic

1. Go to **Amazon SNS** → **Topics** → **Create topic**
2. **Type**: **Standard**
3. **Name**: `job-tracker-alerts`
4. Keep the other defaults → **Create topic**

Create an email subscription:

1. Inside the new topic → **Create subscription**
2. **Protocol**: **Email**
3. **Endpoint**: the admin's email address
4. **Create subscription**
5. Open your inbox → click **Confirm subscription** in the AWS email
6. Refresh the Console → the status becomes **Confirmed**

{{% notice warning %}}
The confirmation step is easy to forget. Until it says **Confirmed**, the alarm can fire but no email will be delivered.
{{% /notice %}}

#### Step 2: Create a CloudWatch Alarm

The alarm warns when API Gateway returns too many 5xx errors.

1. Go to **CloudWatch** → **Alarms** → **All alarms** → **Create alarm**
2. **Select metric** → choose the **ApiGateway** namespace → the **ApiId** dimension
3. Check the row with your **ApiId** and **Metric name = 5xx** → **Select metric**
4. **Statistic**: **Sum**
5. **Period**: **5 minutes**
6. **Conditions**:
   - **Threshold type**: **Static**
   - **Whenever 5xx is**: **Greater/Equal**
   - **than**: `5`
7. **Next** → **Notification**:
   - **Alarm state trigger**: **In alarm**
   - **Send a notification to**: select the `job-tracker-alerts` topic
8. **Next** → **Alarm name**: `api-5xx-alarm` → **Next** → **Create alarm**

![CloudWatch alarm](/images/5-Workshop/5.11-Monitoring-Audit/cloudwatch-alarm.png)

{{% notice info %}}
A threshold of "5 errors in 5 minutes" is sensitive enough to catch real incidents yet tolerant enough to ignore a single stray error. A new alarm starts in **Insufficient data** and turns **OK** after a few minutes.
{{% /notice %}}

If the **5xx** metric does not appear in the list: metrics only materialize after the API has traffic. Send a few requests, then refresh.

#### Step 3: Enable CloudTrail

CloudTrail records **every API call in the account** — not just API Gateway but also Lambda, DynamoDB, S3, Cognito and IAM operations.

1. Go to **AWS CloudTrail** → **Trails** → **Create trail**
2. **Trail name**: `job-tracker-trail`
3. **Storage location**: choose **Create new S3 bucket** (auto-named `aws-cloudtrail-logs-...`)
4. **Log file SSE-KMS encryption**: uncheck (the bucket already uses SSE-S3 by default)
5. **CloudWatch Logs**: skip
6. **Next** → **Event type**: keep **Management events** with both **Read** and **Write** checked
7. **Next** → **Create trail**

Wait a few minutes, then open **Event history** to see recent API calls.

![CloudTrail logging](/images/5-Workshop/5.11-Monitoring-Audit/cloudtrail-logging.png)

Check the S3 bucket → you will see the structure `AWSLogs/<ACCOUNT_ID>/CloudTrail/ap-southeast-1/...` containing `.json.gz` log files.

{{% notice tip %}}
When an incident occurs, Event history lets you trace **who did what, when, and from which IP**. It is a basic but essential audit tool for any production system.
{{% /notice %}}

#### The admin role in this system

The application **deliberately has no in-app admin role**: application data and CVs are sensitive, and least privilege applies to operators as well. No API allows one user to read another user's data.

System administration happens at the **infrastructure layer**:
- **IAM** — access control for AWS resources
- **CloudWatch Alarm + SNS** — incident alerts by email
- **CloudTrail** — traceability of every action

A future extension for aggregate statistics: use **Cognito Groups** — add a user to an `admin` group, the JWT then carries a `cognito:groups` claim, and a Lambda can check it to expose dedicated endpoints (e.g. total users, total jobs) without exposing individual records.
