---
title : "Chi phí"
date : "2026-07-07"
weight : 12
chapter : false
pre : " <b> 5.12 </b> "
---

#### Nội dung
- [Chi phí thực tế](#chi-phí-thực-tế)
- [Phân tích từng dịch vụ](#phân-tích-từng-dịch-vụ)
- [Ước tính khi vận hành thật](#ước-tính-khi-vận-hành-thật)
- [Các quyết định tối ưu chi phí](#các-quyết-định-tối-ưu-chi-phí)

#### Chi phí thực tế

Sau toàn bộ quá trình xây dựng và kiểm thử (hàng nghìn request API, hàng chục lần deploy Lambda, upload CV, gửi email nhắc nhở):

| Hạng mục | Chi phí |
|---|---|
| Backend (Lambda, API Gateway, DynamoDB, S3, Cognito, SES, SNS, SQS, CloudWatch, CloudTrail, KMS) | **$0.85** |
| Amplify Hosting | **$0** (trong free tier) |
| Domain (mua ngoài AWS) | **$0** |
| Route 53 hosted zone | **$0.50/tháng** |
| AWS WAF trên Amplify | **~$23/tháng** |
| **Được cover bởi Free Tier credits** | Toàn bộ |
| **Thực trả** | **$0.00** |

![Billing](/images/5-Workshop/5.12-Cost/billing.png)

![Credits](/images/5-Workshop/5.12-Cost/credits.png)

#### Phân tích từng dịch vụ

| Dịch vụ | Free Tier | Ghi chú |
|---|---|---|
| **Lambda** | 1 triệu request + 400.000 GB-giây/tháng | Dùng arm64 rẻ hơn x86 ~20% |
| **API Gateway (HTTP API)** | 1 triệu request/tháng (12 tháng đầu) | Rẻ hơn REST API ~70% |
| **DynamoDB** | 25 GB storage, On-demand tính theo request | Không Scan → chi phí đọc thấp |
| **S3** | 5 GB storage, 20.000 GET, 2.000 PUT | Bucket Key giảm ~90% gọi KMS |
| **Cognito** | 10.000 MAU miễn phí | Không phát sinh ở quy mô demo |
| **Amplify Hosting** | 1.000 build phút, 15 GB transfer, 5 GB storage/tháng | App tĩnh vài MB, nằm gọn trong free tier |
| **SES** | 200 email/ngày (khi gửi từ Lambda) | Sandbox mode đủ cho demo |
| **CloudWatch** | 10 alarm, 5 GB log/tháng | Alarm 5xx nằm trong hạn mức |
| **CloudTrail** | Management events miễn phí | Chỉ tốn phí lưu log trên S3 (vài KB/ngày) |
| **Route 53** | Không có free tier | $0.50/tháng cho hosted zone |
| **AWS WAF (Amplify)** | Không có free tier | Xem bảng phân tách bên dưới |

#### Phân tách chi phí AWS WAF

Nhiều tài liệu chỉ nhắc con số **$15/tháng** — đó là **phí Amplify cho tính năng firewall**, chưa gồm phí AWS WAF thật sự bên dưới. Chi phí đầy đủ gồm ba khoản:

| Khoản mục | Dịch vụ tính phí | Chi phí |
|---|---|---|
| Amplify Firewall (phí tích hợp) | AWS Amplify | **$15/tháng** |
| WebACL ($5) + 3 managed rules (3 × $1) | AWS WAF | **~$8/tháng** |
| Xử lý request | AWS WAF | **$1.40 / 1 triệu request** |
| **Tổng** | | **~$23–24/tháng** |

Khi bật firewall, Amplify tự tạo một WebACL tên `CreatedByAmplify-<app-id>` với 3 managed rules, scope **CloudFront (Global)**. Giá WAF tiêu chuẩn là $5/tháng cho mỗi WebACL và $1/tháng cho mỗi rule — đó là nguồn gốc của khoản ~$8.

{{% notice tip %}}
Con số ước tính này hiển thị trực tiếp trên màn hình cấu hình firewall của Amplify trước khi bạn bấm **Add firewall**. Luôn đọc kỹ phần "incurs additional costs" ở góc phải dưới.
{{% /notice %}}

#### Ước tính khi vận hành thật

Giả định 1.000 người dùng hoạt động hằng tháng:

| Dịch vụ | Ước tính/tháng |
|---|---|
| Lambda + API Gateway | ~$1–3 |
| DynamoDB On-demand | ~$2–5 |
| S3 (lưu CV) | ~$1–2 |
| Amplify Hosting | ~$0–2 |
| Cognito | $0 (dưới 10.000 MAU) |
| SES + SNS + CloudWatch + CloudTrail | ~$1 |
| Route 53 | $0.50 |
| AWS WAF | ~$23 |
| **Tổng** | **~$28–36/tháng** |

Nếu không bật WAF: **~$5–13/tháng**.

{{% notice info %}}
Kiến trúc serverless cho phép chi phí **tăng tuyến tính theo lượng người dùng thật**, thay vì trả cố định cho server nhàn rỗi 24/7. Một EC2 t3.small chạy liên tục đã tốn ~$15/tháng dù không có ai truy cập.
{{% /notice %}}

#### Các quyết định tối ưu chi phí

| Quyết định | Lý do |
|---|---|
| **HTTP API** thay vì REST API | Rẻ hơn ~70%, đủ tính năng cần thiết |
| **Lambda arm64** thay vì x86 | Rẻ hơn ~20% với cùng hiệu năng |
| **DynamoDB On-demand** | Không cần dự đoán throughput, không trả cho capacity thừa |
| **AWS managed KMS key** (`aws/s3`) thay vì customer managed key | Tiết kiệm $1/tháng cho key |
| **S3 Bucket Key: Enable** | Giảm ~90% số lần gọi KMS |
| **Không dùng VPC** | Tránh chi phí NAT Gateway (~$32/tháng) |
| **Presigned URL** thay vì upload qua Lambda | Giảm thời gian chạy Lambda và data transfer |
| **Không Scan DynamoDB** | Query theo key rẻ hơn Scan nhiều lần |
| **Mua domain ngoài AWS** | Phí đăng ký domain **không được trừ vào promotional credits** |

{{% notice warning %}}
**AWS WAF là khoản chi lớn nhất (~$23/tháng)** trong kiến trúc này. Nếu chỉ triển khai để học tập, hãy **tắt WAF ngay sau khi hoàn thành** để tiết kiệm credits.
{{% /notice %}}
