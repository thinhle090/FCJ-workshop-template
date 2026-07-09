---
title : "Cấu hình API Gateway"
date : "2026-07-07"
weight : 6
chapter : false
pre : " <b> 5.6 </b> "
---

#### Nội dung
- [Bước 1: Tạo HTTP API](#bước-1-tạo-http-api)
- [Bước 2: Tạo Routes](#bước-2-tạo-routes)
- [Bước 3: Tạo JWT Authorizer](#bước-3-tạo-jwt-authorizer)
- [Bước 4: Gắn Integration](#bước-4-gắn-integration)
- [Bước 5: Cấu hình CORS](#bước-5-cấu-hình-cors)
- [Bước 6: Kiểm thử bằng Postman](#bước-6-kiểm-thử-bằng-postman)

{{% notice info %}}
Workshop dùng **HTTP API** thay vì REST API: rẻ hơn khoảng 70%, cấu hình CORS một bước, và JWT authorizer tích hợp Cognito trực tiếp.
{{% /notice %}}

#### Bước 1: Tạo HTTP API

1. Vào **API Gateway** → **Create API**
2. Ở mục **HTTP API**, chọn **Build**
3. **API name**: `job-tracker-api`
4. Bỏ qua phần Integrations → **Next**
5. Bỏ qua Routes → **Next** → Bỏ qua Stages → **Next** → **Create**

Sau khi tạo, ghi lại **Invoke URL** (dạng `https://xxxxx.execute-api.ap-southeast-1.amazonaws.com`) vào `NOTES.md`.

#### Bước 2: Tạo Routes

Menu trái chọn **Routes** → **Create**, tạo lần lượt 6 routes:

| Method | Path |
|---|---|
| GET | `/jobs` |
| POST | `/jobs` |
| GET | `/jobs/{jobId}` |
| PUT | `/jobs/{jobId}` |
| DELETE | `/jobs/{jobId}` |
| POST | `/jobs/{jobId}/cv-url` |

{{% notice warning %}}
Gõ `{jobId}` có ngoặc nhọn và đúng chữ hoa/thường — code Lambda đọc `event.pathParameters.jobId`, sai case sẽ nhận `undefined`.
{{% /notice %}}

#### Bước 3: Tạo JWT Authorizer

1. Menu trái chọn **Authorization** → chọn một route bất kỳ → **Create and attach an authorizer**
2. **Authorizer type**: **JWT**
3. **Name**: `cognito-jwt`
4. **Identity source**: giữ mặc định `$request.header.Authorization`
5. **Issuer URL**: `https://cognito-idp.ap-southeast-1.amazonaws.com/<USER_POOL_ID>`
6. **Audience**: nhập **Client ID** của app client
7. Chọn **Create and attach**

Sau đó gắn authorizer này vào **tất cả 6 routes**: chọn từng route → **Select existing authorizer** → chọn `cognito-jwt` → **Attach authorizer**.

![API Gateway routes](/images/5-Workshop/5.6-APIGateway/apigw-routes.png)

{{% notice warning %}}
Kiểm tra kỹ: cả 6 route đều phải có badge **JWT Auth**. Sót một route là tạo lỗ hổng bảo mật — request không cần token vẫn vào được Lambda.
{{% /notice %}}

#### Bước 4: Gắn Integration

Menu trái chọn **Integrations** → chọn từng route → **Create and attach an integration**:

| Route | Lambda function |
|---|---|
| `GET /jobs` | `job-tracker-get-jobs` |
| `GET /jobs/{jobId}` | `job-tracker-get-jobs` |
| `POST /jobs` | `job-tracker-crud-job` |
| `PUT /jobs/{jobId}` | `job-tracker-crud-job` |
| `DELETE /jobs/{jobId}` | `job-tracker-crud-job` |
| `POST /jobs/{jobId}/cv-url` | `job-tracker-cv-presigned` (tạo ở mục 5.7) |

#### Bước 5: Cấu hình CORS

1. Menu trái chọn **CORS** → **Configure**
2. **Access-Control-Allow-Origin**: nhập `http://localhost:5173` → **Add** (sau khi deploy Amplify sẽ thêm domain thật)
3. **Access-Control-Allow-Headers**: nhập `*` → **Add**
4. **Access-Control-Allow-Methods**: chọn `*`
5. **Access-Control-Max-Age**: `300`
6. **Access-Control-Allow-Credentials**: giữ **NO**
7. **Save**

{{% notice info %}}
Không bật Allow-Credentials vì app gửi JWT qua header `Authorization`, không dùng cookie. Bật YES còn xung đột với Origin `*` theo spec CORS.
{{% /notice %}}

#### Bước 6: Kiểm thử bằng Postman

**Lấy JWT token** — tạo request POST tới `https://cognito-idp.ap-southeast-1.amazonaws.com/`:

- Headers: `Content-Type: application/x-amz-json-1.1`, `X-Amz-Target: AWSCognitoIdentityProviderService.InitiateAuth`
- Body (raw JSON):

```json
{
  "AuthFlow": "USER_PASSWORD_AUTH",
  "ClientId": "<CLIENT_ID>",
  "AuthParameters": {
    "USERNAME": "<email>",
    "PASSWORD": "<password>"
  }
}
```

Trong tab **Scripts → Post-response**, thêm script tự lưu token:

```javascript
const res = pm.response.json();
if (res.AuthenticationResult) {
    pm.environment.set("token", res.AuthenticationResult.IdToken);
}
```

**Kiểm thử các API** — dùng `Authorization: Bearer {{token}}`:

```bash
# 1. Không token → phải nhận 401
GET {{api_url}}/jobs

# 2. Tạo job
POST {{api_url}}/jobs
{
  "company": "FPT Software",
  "position": "Cloud Engineer Intern",
  "followUpDate": "2026-07-08"
}

# 3. Lấy danh sách
GET {{api_url}}/jobs

# 4. Cập nhật
PUT {{api_url}}/jobs/<jobId>
{ "status": "INTERVIEW" }

# 5. Xoá
DELETE {{api_url}}/jobs/<jobId>
```

![Postman 401](/images/5-Workshop/5.6-APIGateway/postman-401.png)

**Checklist hoàn thành:**
- [ ] Gọi API không có token → nhận **401 Unauthorized**
- [ ] POST tạo job → nhận **201**, thấy item trong DynamoDB
- [ ] GET trả đúng danh sách, PUT/DELETE hoạt động
- [ ] User A không đọc/sửa được job của user B

{{% notice tip %}}
Nhớ lấy đúng **IdToken** trong response (không phải `AccessToken`) — chỉ IdToken có claim `aud` khớp với Audience của authorizer.
{{% /notice %}}
