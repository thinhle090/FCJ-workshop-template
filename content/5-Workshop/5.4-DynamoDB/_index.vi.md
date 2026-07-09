---
title : "Tạo bảng DynamoDB"
date : "2026-07-07"
weight : 4
chapter : false
pre : " <b> 5.4 </b> "
---

#### Nội dung
- [Thiết kế dữ liệu](#thiết-kế-dữ-liệu)
- [Bước 1: Tạo bảng](#bước-1-tạo-bảng)
- [Bước 2: Tạo Global Secondary Index](#bước-2-tạo-global-secondary-index)
- [Bước 3: Bật Deletion protection](#bước-3-bật-deletion-protection)

#### Thiết kế dữ liệu

Mỗi item đại diện cho một đơn ứng tuyển:

```json
{
  "userId": "095a85bc-d081-70ec-...",   // partition key, lấy từ claim sub của JWT
  "jobId": "f9b2e10a-c3a1-41e8-...",    // sort key, UUID sinh khi tạo
  "company": "FPT Software",
  "position": "Cloud Engineer Intern",
  "status": "APPLIED",                   // APPLIED | INTERVIEW | OFFER | REJECTED
  "appliedDate": "2026-07-01",
  "followUpDate": "2026-07-08",
  "followUpStatus": "PENDING",           // PENDING | NOTIFIED | NONE
  "notes": "Nộp qua career site",
  "cvKey": "userId/jobId/cv.pdf",
  "userEmail": "user@example.com",
  "createdAt": "2026-07-01T10:00:00Z",
  "updatedAt": "2026-07-01T10:00:00Z"
}
```

{{% notice info %}}
Thiết kế partition key là `userId` giúp mọi truy vấn của người dùng chỉ đọc đúng dữ liệu của họ bằng **Query**, không bao giờ cần **Scan** toàn bảng.
{{% /notice %}}

#### Bước 1: Tạo bảng

1. Vào **DynamoDB** → **Create table**
2. **Table name**: `JobTrackerJobs`
3. **Partition key**: `userId` — kiểu **String**
4. **Sort key**: `jobId` — kiểu **String**
5. **Table settings**: chọn **Customize settings**
6. **Table class**: DynamoDB Standard
7. **Read/write capacity settings**: chọn **On-demand** (trả theo request, không cần dự đoán throughput)
8. **Encryption at rest**: chọn **Owned by Amazon DynamoDB** hoặc **AWS managed key** (`aws/dynamodb`)
9. Chọn **Create table**

![DynamoDB table](/images/5-Workshop/5.4-DynamoDB/dynamodb-table.png)

#### Bước 2: Tạo Global Secondary Index

GSI này phục vụ luồng nhắc nhở tự động (mục 5.10) — cho phép tìm nhanh các job đến hạn mà không Scan.

1. Mở bảng `JobTrackerJobs` → tab **Indexes** → **Create index**
2. **Partition key**: `followUpStatus` — kiểu **String**
3. **Sort key**: `followUpDate` — kiểu **String**
4. **Index name**: `FollowUpIndex`
5. **Attribute projections**: chọn **All**
6. Chọn **Create index**, đợi trạng thái chuyển **Active**

{{% notice warning %}}
**Key của GSI không thể sửa sau khi tạo.** Gõ chính xác `followUpStatus` và `followUpDate` (phân biệt hoa/thường). Nếu sai, Lambda sẽ báo lỗi `ValidationException: Query condition missed key schema element` và bạn phải xoá index rồi tạo lại (dữ liệu trong bảng không bị ảnh hưởng).
{{% /notice %}}

#### Bước 3: Bật Deletion protection

1. Mở bảng → tab **Additional settings** (hoặc **Table details**)
2. Tìm **Deletion protection** → **Turn on**

Thao tác 10 giây này tránh được thảm hoạ ai đó lỡ tay xoá bảng.
