---
title : "Tạo Lambda xử lý Job"
date : "2026-07-07"
weight : 5
chapter : false
pre : " <b> 5.5 </b> "
---

#### Nội dung
- [Bước 1: Tạo Lambda CRUD Job](#bước-1-tạo-lambda-crud-job)
- [Bước 2: Cấp quyền IAM](#bước-2-cấp-quyền-iam)
- [Bước 3: Tạo Lambda Get Jobs](#bước-3-tạo-lambda-get-jobs)
- [Nguyên tắc bảo mật trong code](#nguyên-tắc-bảo-mật-trong-code)

#### Bước 1: Tạo Lambda CRUD Job

1. Vào **Lambda** → **Create function** → **Author from scratch**
2. **Function name**: `job-tracker-crud-job`
3. **Runtime**: **Node.js 20.x**
4. **Architecture**: **arm64** (rẻ hơn x86 khoảng 20%)
5. Chọn **Create function**

Sau khi tạo, ở tab **Code**, xoá nội dung `index.mjs` và dán code sau:

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

// Lấy userId + email từ JWT đã được API Gateway xác thực
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
        return res(400, { message: "company và position là bắt buộc" });
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
      if (!jobId) return res(400, { message: "Thiếu jobId" });
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
      // Đổi followUpDate thì reset trạng thái nhắc nhở về PENDING
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
      if (!jobId) return res(400, { message: "Thiếu jobId" });
      await ddb.send(new DeleteCommand({
        TableName: TABLE,
        Key: { userId, jobId },
        ConditionExpression: "attribute_exists(jobId)",
      }));
      return res(200, { message: "Đã xoá", jobId });
    }

    return res(405, { message: `Method ${method} không hỗ trợ` });
  } catch (err) {
    if (err.name === "ConditionalCheckFailedException") {
      return res(404, { message: "Job không tồn tại hoặc không thuộc về bạn" });
    }
    console.error(err);
    return res(500, { message: "Lỗi server", error: err.message });
  }
};
```

Chọn **Deploy** để lưu code.

Tiếp theo, thêm biến môi trường:
1. Tab **Configuration** → **Environment variables** → **Edit** → **Add environment variable**
2. Key: `TABLE_NAME`, Value: `JobTrackerJobs` → **Save**

#### Bước 2: Cấp quyền IAM

1. Tab **Configuration** → **Permissions** → click vào **Role name** (mở IAM console)
2. **Add permissions** → **Create inline policy** → tab **JSON**
3. Dán policy sau (thay `<ACCOUNT_ID>` bằng Account ID 12 số của bạn):

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

#### Bước 3: Tạo Lambda Get Jobs

Lặp lại Bước 1 với tên `job-tracker-get-jobs`, dán code sau:

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
    // GET /jobs/{jobId} — chi tiết một job
    if (jobId) {
      const out = await ddb.send(new GetCommand({
        TableName: TABLE,
        Key: { userId, jobId },
      }));
      if (!out.Item) return res(404, { message: "Không tìm thấy job" });
      return res(200, out.Item);
    }

    // GET /jobs — Query theo partition key, KHÔNG Scan
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
    return res(500, { message: "Lỗi server", error: err.message });
  }
};
```

Cấu hình tương tự: env `TABLE_NAME=JobTrackerJobs`, và inline policy với quyền tối thiểu:

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
Lambda đọc chỉ cần `Query` và `GetItem`, không cần `PutItem`/`DeleteItem` — đây là nguyên tắc **least privilege**: mỗi function chỉ có đúng quyền nó cần.
{{% /notice %}}

#### Nguyên tắc bảo mật trong code

| Nguyên tắc | Cách thực hiện |
|---|---|
| Không tin dữ liệu client | `userId` luôn lấy từ `event.requestContext.authorizer.jwt.claims.sub` |
| Không sửa/xoá job người khác | Key là `{userId, jobId}` + `ConditionExpression: attribute_exists(jobId)` |
| Không Scan toàn bảng | Luôn dùng `QueryCommand` với `KeyConditionExpression` |
| Least privilege | Mỗi Lambda một inline policy riêng, chỉ đủ quyền cần thiết |
