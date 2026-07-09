---
title : "Lưu trữ CV với S3 và Presigned URL"
date : "2026-07-07"
weight : 7
chapter : false
pre : " <b> 5.7 </b> "
---

#### Nội dung
- [Cơ chế Presigned URL](#cơ-chế-presigned-url)
- [Bước 1: Tạo S3 bucket riêng tư](#bước-1-tạo-s3-bucket-riêng-tư)
- [Bước 2: Cấu hình CORS cho bucket](#bước-2-cấu-hình-cors-cho-bucket)
- [Bước 3: Tạo Lambda cấp Presigned URL](#bước-3-tạo-lambda-cấp-presigned-url)
- [Bước 4: Kiểm thử](#bước-4-kiểm-thử)

#### Cơ chế Presigned URL

Thay vì upload file qua Lambda (tốn thời gian chạy, giới hạn 6MB payload), hệ thống dùng **Presigned URL**:

1. Client gọi API xin URL, kèm JWT
2. Lambda xác minh job thuộc về user, rồi **ký một URL S3 có thời hạn 5 phút**
3. Client dùng URL đó **upload/download trực tiếp với S3**, không qua Lambda

Bucket vẫn **private tuyệt đối** — chỉ URL đã ký mới truy cập được.

#### Bước 1: Tạo S3 bucket riêng tư

1. Vào **S3** → **Create bucket**
2. **Bucket name**: `job-tracker-cv-<tên-nhóm>-2026` (chỉ chữ thường, số, dấu gạch ngang)
3. **Block Public Access settings**: giữ nguyên **Block all public access** (tick đủ 4 mục)
4. **Default encryption**: chọn **Server-side encryption with AWS KMS keys (SSE-KMS)**
5. **AWS KMS key**: chọn **Choose from your AWS KMS keys** → chọn `aws/s3` (AWS managed key, miễn phí)
6. **Bucket Key**: **Enable** (giảm ~90% số lần gọi KMS, tiết kiệm chi phí)
7. Chọn **Create bucket**

{{% notice info %}}
Đừng tạo **customer managed key** — key đó tốn $1/tháng. AWS managed key `aws/s3` miễn phí và vẫn thể hiện đầy đủ vai trò của KMS trong kiến trúc.
{{% /notice %}}

#### Bước 2: Cấu hình CORS cho bucket

Cho phép browser upload trực tiếp:

1. Mở bucket → tab **Permissions** → kéo xuống **Cross-origin resource sharing (CORS)** → **Edit**
2. Dán cấu hình:

```json
[
  {
    "AllowedHeaders": ["*"],
    "AllowedMethods": ["PUT", "GET"],
    "AllowedOrigins": ["http://localhost:5173"],
    "ExposeHeaders": ["ETag"]
  }
]
```

3. **Save changes** (sau khi deploy Amplify sẽ thêm domain thật vào `AllowedOrigins`)

#### Bước 3: Tạo Lambda cấp Presigned URL

Tạo function `job-tracker-cv-presigned` (Node.js 20.x, arm64) với code:

```javascript
import { S3Client, PutObjectCommand, GetObjectCommand } from "@aws-sdk/client-s3";
import { getSignedUrl } from "@aws-sdk/s3-request-presigner";
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient, GetCommand, UpdateCommand } from "@aws-sdk/lib-dynamodb";

const s3 = new S3Client({});
const ddb = DynamoDBDocumentClient.from(new DynamoDBClient({}));
const TABLE = process.env.TABLE_NAME;
const BUCKET = process.env.BUCKET_NAME;
const URL_TTL = 300; // URL sống 5 phút

const res = (statusCode, body) => ({
  statusCode,
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify(body),
});

export const handler = async (event) => {
  const userId = event.requestContext?.authorizer?.jwt?.claims?.sub;
  if (!userId) return res(401, { message: "Unauthorized" });

  const jobId = event.pathParameters?.jobId;
  if (!jobId) return res(400, { message: "Thiếu jobId" });

  try {
    // Job phải tồn tại và thuộc về user này
    const job = await ddb.send(new GetCommand({ TableName: TABLE, Key: { userId, jobId } }));
    if (!job.Item) return res(404, { message: "Không tìm thấy job" });

    const b = JSON.parse(event.body || "{}");

    // ---------- upload: cấp URL PUT ----------
    if (b.mode === "upload") {
      const fileName = (b.fileName || "cv.pdf").replace(/[^a-zA-Z0-9._-]/g, "_");
      if (!fileName.toLowerCase().endsWith(".pdf")) {
        return res(400, { message: "Chỉ nhận file PDF" });
      }
      // Key theo userId/jobId → mỗi user một "ngăn" riêng
      const key = `${userId}/${jobId}/${fileName}`;
      const url = await getSignedUrl(
        s3,
        new PutObjectCommand({ Bucket: BUCKET, Key: key, ContentType: "application/pdf" }),
        { expiresIn: URL_TTL },
      );
      await ddb.send(new UpdateCommand({
        TableName: TABLE,
        Key: { userId, jobId },
        UpdateExpression: "SET cvKey = :k, updatedAt = :t",
        ExpressionAttributeValues: { ":k": key, ":t": new Date().toISOString() },
      }));
      return res(200, { uploadUrl: url, key, expiresIn: URL_TTL });
    }

    // ---------- download: cấp URL GET ----------
    if (b.mode === "download") {
      if (!job.Item.cvKey) return res(404, { message: "Job này chưa có CV" });
      const url = await getSignedUrl(
        s3,
        new GetObjectCommand({ Bucket: BUCKET, Key: job.Item.cvKey }),
        { expiresIn: URL_TTL },
      );
      return res(200, { downloadUrl: url, expiresIn: URL_TTL });
    }

    return res(400, { message: 'mode phải là "upload" hoặc "download"' });
  } catch (err) {
    console.error(err);
    return res(500, { message: "Lỗi server", error: err.message });
  }
};
```

**Environment variables:**
- `TABLE_NAME` = `JobTrackerJobs`
- `BUCKET_NAME` = tên bucket vừa tạo

**IAM inline policy** (thay `<ACCOUNT_ID>` và `<BUCKET_NAME>`):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:PutObject", "s3:GetObject"],
      "Resource": "arn:aws:s3:::<BUCKET_NAME>/*"
    },
    {
      "Effect": "Allow",
      "Action": ["kms:GenerateDataKey", "kms:Decrypt"],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": ["dynamodb:GetItem", "dynamodb:UpdateItem"],
      "Resource": "arn:aws:dynamodb:ap-southeast-1:<ACCOUNT_ID>:table/JobTrackerJobs"
    }
  ]
}
```

Cuối cùng, quay lại API Gateway gắn function này vào route `POST /jobs/{jobId}/cv-url`.

#### Bước 4: Kiểm thử

**Xin URL upload** (Postman, có Bearer token):

```bash
POST {{api_url}}/jobs/{{job_id}}/cv-url
{ "mode": "upload", "fileName": "cv.pdf" }
```

**Upload file lên S3** — request PUT tới `uploadUrl` nhận được:
- Authorization: **No Auth** (chữ ký đã nằm trong URL)
- Headers: `Content-Type: application/pdf`
- Body: **binary** → chọn file PDF

**Xin URL download**:

```bash
POST {{api_url}}/jobs/{{job_id}}/cv-url
{ "mode": "download" }
```

Mở `downloadUrl` trong browser → file PDF hiện ra.

![CV uploaded](/images/5-Workshop/5.7-CV-S3-Presigned/s3-cv-uploaded.png)

**Kiểm chứng bảo mật:** vào S3 Console → click file CV → copy **Object URL** (URL trần không có chữ ký) → mở trong browser → nhận **AccessDenied**.

![AccessDenied](/images/5-Workshop/5.7-CV-S3-Presigned/s3-access-denied.png)

{{% notice warning %}}
Khi test upload bằng Postman, phải tắt Bearer token cho request PUT lên S3 (hai cơ chế auth cùng lúc bị S3 từ chối) và header `Content-Type` phải khớp đúng giá trị đã ký, nếu không sẽ nhận `403 SignatureDoesNotMatch`.
{{% /notice %}}
