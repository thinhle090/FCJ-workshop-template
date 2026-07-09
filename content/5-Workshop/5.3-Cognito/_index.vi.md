---
title : "Cấu hình Amazon Cognito"
date : "2026-07-07"
weight : 3
chapter : false
pre : " <b> 5.3 </b> "
---

#### Nội dung
- [Bước 1: Tạo User Pool](#bước-1-tạo-user-pool)
- [Bước 2: Ghi lại thông tin quan trọng](#bước-2-ghi-lại-thông-tin-quan-trọng)
- [Bước 3: Bật auth flow cho kiểm thử](#bước-3-bật-auth-flow-cho-kiểm-thử)
- [Bước 4: Tạo user test](#bước-4-tạo-user-test)

{{% notice info %}}
Đảm bảo region đang chọn là **ap-southeast-1 (Singapore)** — kiểm tra ở góc phải trên Console trước mỗi bước.
{{% /notice %}}

#### Bước 1: Tạo User Pool

1. Vào **Amazon Cognito** → **Create user pool**
2. **Application type**: chọn **Single-page application (SPA)**
3. **Name your application**: `job-tracker-web`
4. **Options for sign-in identifiers**: tick **Email**
5. **Required attributes for sign-up**: chọn **email**
6. Chọn **Create user directory**

![Cognito User Pool](/images/5-Workshop/5.3-Cognito/cognito-pool.png)

#### Bước 2: Ghi lại thông tin quan trọng

Sau khi tạo xong, ghi 3 giá trị này vào `NOTES.md` — sẽ dùng suốt dự án:

| Thông tin | Vị trí |
|---|---|
| **User Pool ID** | Trang tổng quan user pool (dạng `ap-southeast-1_xxxxxxx`) |
| **Client ID** | Tab **App clients** → chọn client vừa tạo |
| **Region** | `ap-southeast-1` |

#### Bước 3: Bật auth flow cho kiểm thử

Để lấy JWT token bằng Postman mà không cần frontend:

1. Vào user pool → tab **App clients** → chọn client → **Edit**
2. Trong **Authentication flows**, tick thêm **`ALLOW_USER_PASSWORD_AUTH`**
3. Chọn **Save changes**

#### Bước 4: Tạo user test

1. Vào tab **Users** → **Create user**
2. Nhập email của bạn, đặt password
3. Tick **Mark email address as verified**
4. Chọn **Create user**

Nếu user ở trạng thái `FORCE_CHANGE_PASSWORD`, chạy lệnh sau trong **CloudShell** để chốt password:

```bash
aws cognito-idp admin-set-user-password \
  --user-pool-id <USER_POOL_ID> \
  --username <email> \
  --password '<mật khẩu>' \
  --permanent \
  --region ap-southeast-1
```

{{% notice tip %}}
**IdToken vs AccessToken** — Sau này khi cấu hình JWT authorizer với `Audience = Client ID`, chỉ **IdToken** mới chứa claim `aud` khớp. Dùng nhầm AccessToken sẽ nhận lỗi 401 dù token hoàn toàn hợp lệ. Đây là lỗi rất phổ biến khi test bằng Postman.
{{% /notice %}}
