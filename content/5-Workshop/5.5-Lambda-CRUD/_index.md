---
title : "Create the Job Lambda Functions"
date : "2026-07-07"
weight : 5
chapter : false
pre : " <b> 5.5 </b> "
---

#### Content
- [Step 1: Create the CRUD Job Lambda](#step-1-create-the-crud-job-lambda)
- [Step 2: Grant IAM permissions](#step-2-grant-iam-permissions)
- [Step 3: Create the Get Jobs Lambda](#step-3-create-the-get-jobs-lambda)
- [Security principles in the code](#security-principles-in-the-code)

#### Step 1: Create the CRUD Job Lambda

1. Go to **Lambda** → **Create function** → **Author from scratch**
2. **Function name**: `job-tracker-crud-job`
3. **Runtime**: **Node.js 20.x**
4. **Architecture**: **arm64** (about 20% cheaper than x86)
5. Choose **Create function**

In the **Code** tab, replace the contents of `index.mjs` with:

```javascript
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import {
  DynamoDBDocumentClient, PutCommand, UpdateCommand, DeleteCommand,
} from "@aws-sdk/lib-dynamodb";
import { randomUUID } from "crypto";

const ddb = DynamoDBDocumentClient.from(new DynamoDBClient({}));
const TABLE = process.env.TABLE_NAME;

const res = (statusCode, body) => ({
  statusCode,
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify(body),
});

// Read userId + email from the JWT already validated by API Gateway
const getClaims = (event) => {
  const c = event.requestContext?.authorizer?.jwt?.claims || {};
  return { userId: c.sub, email: c.email };
};

export const handler = async (event) => {
  const { userId, email } = getClaims(event);
  if (!userId) return res(401, { message: "Unauthorized" });

  const method = event.requestContext?.http?.method;
  const jobId = event.pathParameters?.jobId;
  const now = new Date().toISOString();

  try {
    // ---------- POST /jobs ----------
    if (method === "POST") {
      const b = JSON.parse(event.body || "{}");
      if (!b.company || !b.position) {
        return res(400, { message: "company and position are required" });
      }
      const item = {
        userId,
        jobId: randomUUID(),
        company: b.company,
        position: b.position,
        status: b.status || "APPLIED",
        appliedDate: b.appliedDate || now.slice(0, 10),
        followUpDate: b.followUpDate || null,
        followUpStatus: b.followUpDate ? "PENDING" : "NONE",
        notes: b.notes || "",
        cvKey: null,
        userEmail: email,
        createdAt: now,
        updatedAt: now,
      };
      await ddb.send(new PutCommand({ TableName: TABLE, Item: item }));
      return res(201, item);
    }

    // ---------- PUT /jobs/{jobId} ----------
    if (method === "PUT") {
      if (!jobId) return res(400, { message: "Missing jobId" });
      const b = JSON.parse(event.body || "{}");
      const allowed = ["company", "position", "status", "appliedDate", "followUpDate", "notes"];
      const sets = ["updatedAt = :updatedAt"];
      const vals = { ":updatedAt": now };
      const names = {};
      for (const k of allowed) {
        if (b[k] !== undefined) {
          sets.push(`#${k} = :${k}`);
          names[`#${k}`] = k;
          vals[`:${k}`] = b[k];
        }
      }
      // Changing followUpDate resets the reminder status to PENDING
      if (b.followUpDate !== undefined) {
        sets.push("#fus = :fus");
        names["#fus"] = "followUpStatus";
        vals[":fus"] = b.followUpDate ? "PENDING" : "NONE";
      }
      const out = await ddb.send(new UpdateCommand({
        TableName: TABLE,
        Key: { userId, jobId },
        UpdateExpression: "SET " + sets.join(", "),
        ExpressionAttributeNames: Object.keys(names).length ? names : undefined,
        ExpressionAttributeValues: vals,
        ConditionExpression: "attribute_exists(jobId)",
        ReturnValues: "ALL_NEW",
      }));
      return res(200, out.Attributes);
    }

    // ---------- DELETE /jobs/{jobId} ----------
    if (method === "DELETE") {
      if (!jobId) return res(400, { message: "Missing jobId" });
      await ddb.send(new DeleteCommand({
        TableName: TABLE,
        Key: { userId, jobId },
        ConditionExpression: "attribute_exists(jobId)",
      }));
      return res(200, { message: "Deleted", jobId });
    }

    return res(405, { message: `Method ${method} not supported` });
  } catch (err) {
    if (err.name === "ConditionalCheckFailedException") {
      return res(404, { message: "Job not found or does not belong to you" });
    }
    console.error(err);
    return res(500, { message: "Server error", error: err.message });
  }
};
```

Choose **Deploy** to save.

Then add the environment variable:
1. **Configuration** tab → **Environment variables** → **Edit** → **Add environment variable**
2. Key: `TABLE_NAME`, Value: `JobTrackerJobs` → **Save**

#### Step 2: Grant IAM permissions

1. **Configuration** tab → **Permissions** → click the **Role name** (opens IAM console)
2. **Add permissions** → **Create inline policy** → **JSON** tab
3. Paste this policy (replace `<ACCOUNT_ID>` with your 12-digit Account ID):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:PutItem",
        "dynamodb:UpdateItem",
        "dynamodb:DeleteItem"
      ],
      "Resource": "arn:aws:dynamodb:ap-southeast-1:<ACCOUNT_ID>:table/JobTrackerJobs"
    }
  ]
}
```

4. **Next** → Policy name: `job-tracker-crud-policy` → **Create policy**

#### Step 3: Create the Get Jobs Lambda

Repeat Step 1 with the name `job-tracker-get-jobs` and this code:

```javascript
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient, QueryCommand, GetCommand } from "@aws-sdk/lib-dynamodb";

const ddb = DynamoDBDocumentClient.from(new DynamoDBClient({}));
const TABLE = process.env.TABLE_NAME;

const res = (statusCode, body) => ({
  statusCode,
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify(body),
});

export const handler = async (event) => {
  const userId = event.requestContext?.authorizer?.jwt?.claims?.sub;
  if (!userId) return res(401, { message: "Unauthorized" });

  const jobId = event.pathParameters?.jobId;

  try {
    // GET /jobs/{jobId} — a single job
    if (jobId) {
      const out = await ddb.send(new GetCommand({
        TableName: TABLE,
        Key: { userId, jobId },
      }));
      if (!out.Item) return res(404, { message: "Job not found" });
      return res(200, out.Item);
    }

    // GET /jobs — Query by partition key, never a Scan
    const qs = event.queryStringParameters || {};
    const params = {
      TableName: TABLE,
      KeyConditionExpression: "userId = :uid",
      ExpressionAttributeValues: { ":uid": userId },
    };
    if (qs.status) {
      params.FilterExpression = "#st = :st";
      params.ExpressionAttributeNames = { "#st": "status" };
      params.ExpressionAttributeValues[":st"] = qs.status;
    }
    const out = await ddb.send(new QueryCommand(params));
    return res(200, { count: out.Count, items: out.Items });
  } catch (err) {
    console.error(err);
    return res(500, { message: "Server error", error: err.message });
  }
};
```

Configure it the same way: env `TABLE_NAME=JobTrackerJobs`, plus a minimal inline policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["dynamodb:Query", "dynamodb:GetItem"],
      "Resource": "arn:aws:dynamodb:ap-southeast-1:<ACCOUNT_ID>:table/JobTrackerJobs"
    }
  ]
}
```

{{% notice tip %}}
The read Lambda only needs `Query` and `GetItem` — no `PutItem`/`DeleteItem`. This is **least privilege**: each function gets exactly the permissions it needs and nothing more.
{{% /notice %}}

#### Security principles in the code

| Principle | Implementation |
|---|---|
| Never trust client data | `userId` always comes from `event.requestContext.authorizer.jwt.claims.sub` |
| Cannot touch another user's job | Key is `{userId, jobId}` + `ConditionExpression: attribute_exists(jobId)` |
| Never Scan the table | Always `QueryCommand` with a `KeyConditionExpression` |
| Least privilege | One inline policy per Lambda with only the required actions |
