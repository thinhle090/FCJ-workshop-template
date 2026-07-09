---
title : "Create the DynamoDB Table"
date : "2026-07-07"
weight : 4
chapter : false
pre : " <b> 5.4 </b> "
---

#### Content
- [Data design](#data-design)
- [Step 1: Create the table](#step-1-create-the-table)
- [Step 2: Create the Global Secondary Index](#step-2-create-the-global-secondary-index)
- [Step 3: Enable Deletion protection](#step-3-enable-deletion-protection)

#### Data design

Each item represents one job application:

```json
{
  "userId": "095a85bc-d081-70ec-...",   // partition key, from the JWT sub claim
  "jobId": "f9b2e10a-c3a1-41e8-...",    // sort key, UUID generated on creation
  "company": "FPT Software",
  "position": "Cloud Engineer Intern",
  "status": "APPLIED",                   // APPLIED | INTERVIEW | OFFER | REJECTED
  "appliedDate": "2026-07-01",
  "followUpDate": "2026-07-08",
  "followUpStatus": "PENDING",           // PENDING | NOTIFIED | NONE
  "notes": "Applied via careers site",
  "cvKey": "userId/jobId/cv.pdf",
  "userEmail": "user@example.com",
  "createdAt": "2026-07-01T10:00:00Z",
  "updatedAt": "2026-07-01T10:00:00Z"
}
```

{{% notice info %}}
Using `userId` as the partition key means every user query reads only that user's data via **Query** — a full-table **Scan** is never required.
{{% /notice %}}

#### Step 1: Create the table

1. Go to **DynamoDB** → **Create table**
2. **Table name**: `JobTrackerJobs`
3. **Partition key**: `userId` — type **String**
4. **Sort key**: `jobId` — type **String**
5. **Table settings**: choose **Customize settings**
6. **Table class**: DynamoDB Standard
7. **Read/write capacity settings**: choose **On-demand** (pay per request, no throughput planning)
8. **Encryption at rest**: choose **Owned by Amazon DynamoDB** or **AWS managed key** (`aws/dynamodb`)
9. Choose **Create table**

![DynamoDB table](/images/5-Workshop/5.4-DynamoDB/dynamodb-table.png)

#### Step 2: Create the Global Secondary Index

This GSI powers the automatic reminder flow (section 5.10) — it finds due jobs quickly without a Scan.

1. Open the `JobTrackerJobs` table → **Indexes** tab → **Create index**
2. **Partition key**: `followUpStatus` — type **String**
3. **Sort key**: `followUpDate` — type **String**
4. **Index name**: `FollowUpIndex`
5. **Attribute projections**: choose **All**
6. Choose **Create index** and wait for the status to become **Active**

{{% notice warning %}}
**GSI keys cannot be changed after creation.** Type `followUpStatus` and `followUpDate` exactly (case-sensitive). If they are wrong, the Lambda will fail with `ValidationException: Query condition missed key schema element` and you will have to delete and recreate the index (table data is unaffected).
{{% /notice %}}

#### Step 3: Enable Deletion protection

1. Open the table → **Additional settings** (or **Table details**)
2. Find **Deletion protection** → **Turn on**

This 10-second action prevents someone from accidentally deleting the table.
