---
title : "CV Storage with S3 and Presigned URLs"
date : "2026-07-07"
weight : 7
chapter : false
pre : " <b> 5.7 </b> "
---

#### Content
- [How Presigned URLs work](#how-presigned-urls-work)
- [Step 1: Create a private S3 bucket](#step-1-create-a-private-s3-bucket)
- [Step 2: Configure bucket CORS](#step-2-configure-bucket-cors)
- [Step 3: Create the Presigned URL Lambda](#step-3-create-the-presigned-url-lambda)
- [Step 4: Test](#step-4-test)

#### How Presigned URLs work

Instead of uploading files through Lambda (which costs execution time and caps payloads at 6MB), the system uses **Presigned URLs**:

1. The client requests a URL from the API, sending its JWT
2. The Lambda verifies the job belongs to that user, then **signs an S3 URL valid for 5 minutes**
3. The client uses that URL to **upload/download directly to S3**, bypassing Lambda

The bucket stays **completely private** — only signed URLs can access it.

#### Step 1: Create a private S3 bucket

1. Go to **S3** → **Create bucket**
2. **Bucket name**: `job-tracker-cv-<team-name>-2026` (lowercase letters, digits, hyphens only)
3. **Block Public Access settings**: keep **Block all public access** (all four boxes checked)
4. **Default encryption**: choose **Server-side encryption with AWS KMS keys (SSE-KMS)**
5. **AWS KMS key**: choose **Choose from your AWS KMS keys** → select `aws/s3` (AWS managed key, free)
6. **Bucket Key**: **Enable** (cuts KMS calls by ~90%, saving cost)
7. Choose **Create bucket**

{{% notice info %}}
Do not create a **customer managed key** — it costs $1/month. The AWS managed key `aws/s3` is free and still demonstrates KMS in the architecture.
{{% /notice %}}

#### Step 2: Configure bucket CORS

Allow the browser to upload directly:

1. Open the bucket → **Permissions** tab → scroll to **Cross-origin resource sharing (CORS)** → **Edit**
2. Paste:

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

3. **Save changes** (the real domain gets added to `AllowedOrigins` after the Amplify deployment)

#### Step 3: Create the Presigned URL Lambda

Create the function `job-tracker-cv-presigned` (Node.js 20.x, arm64) with this code:

```javascript
import { S3Client, PutObjectCommand, GetObjectCommand } from "@aws-sdk/client-s3";
import { getSignedUrl } from "@aws-sdk/s3-request-presigner";
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient, GetCommand, UpdateCommand } from "@aws-sdk/lib-dynamodb";

const s3 = new S3Client({});
const ddb = DynamoDBDocumentClient.from(new DynamoDBClient({}));
const TABLE = process.env.TABLE_NAME;
const BUCKET = process.env.BUCKET_NAME;
const URL_TTL = 300; // URL valid for 5 minutes

const res = (statusCode, body) => ({
  statusCode,
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify(body),
});

export const handler = async (event) => {
  const userId = event.requestContext?.authorizer?.jwt?.claims?.sub;
  if (!userId) return res(401, { message: "Unauthorized" });

  const jobId = event.pathParameters?.jobId;
  if (!jobId) return res(400, { message: "Missing jobId" });

  try {
    // The job must exist and belong to this user
    const job = await ddb.send(new GetCommand({ TableName: TABLE, Key: { userId, jobId } }));
    if (!job.Item) return res(404, { message: "Job not found" });

    const b = JSON.parse(event.body || "{}");

    // ---------- upload: issue a PUT URL ----------
    if (b.mode === "upload") {
      const fileName = (b.fileName || "cv.pdf").replace(/[^a-zA-Z0-9._-]/g, "_");
      if (!fileName.toLowerCase().endsWith(".pdf")) {
        return res(400, { message: "Only PDF files are accepted" });
      }
      // Key is userId/jobId → each user gets a private compartment
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

    // ---------- download: issue a GET URL ----------
    if (b.mode === "download") {
      if (!job.Item.cvKey) return res(404, { message: "This job has no CV yet" });
      const url = await getSignedUrl(
        s3,
        new GetObjectCommand({ Bucket: BUCKET, Key: job.Item.cvKey }),
        { expiresIn: URL_TTL },
      );
      return res(200, { downloadUrl: url, expiresIn: URL_TTL });
    }

    return res(400, { message: 'mode must be "upload" or "download"' });
  } catch (err) {
    console.error(err);
    return res(500, { message: "Server error", error: err.message });
  }
};
```

**Environment variables:**
- `TABLE_NAME` = `JobTrackerJobs`
- `BUCKET_NAME` = the bucket you just created

**IAM inline policy** (replace `<ACCOUNT_ID>` and `<BUCKET_NAME>`):

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

Finally, return to API Gateway and attach this function to the `POST /jobs/{jobId}/cv-url` route.

#### Step 4: Test

**Request an upload URL** (Postman, with the Bearer token):

```bash
POST {{api_url}}/jobs/{{job_id}}/cv-url
{ "mode": "upload", "fileName": "cv.pdf" }
```

**Upload the file to S3** — a PUT request to the returned `uploadUrl`:
- Authorization: **No Auth** (the signature is already in the URL)
- Headers: `Content-Type: application/pdf`
- Body: **binary** → select a PDF file

**Request a download URL**:

```bash
POST {{api_url}}/jobs/{{job_id}}/cv-url
{ "mode": "download" }
```

Open the `downloadUrl` in a browser → the PDF appears.

![CV uploaded](/images/5-Workshop/5.7-CV-S3-Presigned/s3-cv-uploaded.png)

**Security verification:** in the S3 Console → click the CV file → copy the **Object URL** (unsigned) → open it in a browser → you get **AccessDenied**.

![AccessDenied](/images/5-Workshop/5.7-CV-S3-Presigned/s3-access-denied.png)

{{% notice warning %}}
When testing the upload in Postman, disable the Bearer token on the PUT request to S3 (two auth mechanisms at once are rejected) and make sure the `Content-Type` header matches the signed value exactly, otherwise you get `403 SignatureDoesNotMatch`.
{{% /notice %}}
