---
title : "Giới thiệu"
date : "2026-07-07"
weight : 1
chapter : false
pre : " <b> 5.1 </b> "
---

#### Kiến trúc serverless

- **Serverless** là mô hình xây dựng ứng dụng mà nhà phát triển không cần quản lý máy chủ. AWS chịu trách nhiệm cấp phát, mở rộng và vá lỗi hạ tầng bên dưới.
- Các dịch vụ serverless như **AWS Lambda**, **Amazon DynamoDB**, **Amazon S3** hay **Amazon API Gateway** tính tiền theo mức sử dụng thực tế — không có chi phí cho tài nguyên nhàn rỗi.
- **Amazon Cognito** cung cấp xác thực người dùng và cấp **JWT token**, cho phép API Gateway kiểm tra danh tính người gọi mà không cần viết logic xác thực.

#### Bài toán

Người tìm việc thường nộp hàng chục đơn ứng tuyển cùng lúc, dễ quên thời điểm follow-up với nhà tuyển dụng và khó theo dõi CV nào đã gửi cho công ty nào.

**Giải pháp:** một ứng dụng web cho phép quản lý tập trung các đơn ứng tuyển, đính kèm CV riêng cho từng đơn, và tự động nhận email nhắc nhở khi đến hạn follow-up.

#### Tổng quan workshop

Trong workshop này, bạn sẽ xây dựng hệ thống gồm hai phần:

- **Frontend** — ứng dụng React (Vite + TypeScript + Tailwind) được build và phân phối qua **AWS Amplify Hosting** với CDN toàn cầu, HTTPS tự động và CI/CD từ GitHub. **AWS WAF** lọc traffic độc hại ngay tại edge, **Amazon Route 53** quản lý DNS cho domain riêng.

- **Backend** — toàn bộ chạy trên các dịch vụ serverless tại region **ap-southeast-1 (Singapore)**: Cognito xác thực người dùng, API Gateway (HTTP API) làm điểm vào có JWT authorizer, bốn hàm Lambda xử lý logic, DynamoDB lưu dữ liệu job, S3 và KMS lưu trữ CV được mã hoá. EventBridge kích hoạt Lambda quét job đến hạn mỗi sáng và gửi email qua SES, lỗi được đưa vào SQS Dead Letter Queue. CloudWatch Alarm cảnh báo admin qua SNS, CloudTrail ghi log audit vào S3.

![Kiến trúc hệ thống](/images/5-Workshop/5.1-Introduction/architecture.png)

#### Luồng hoạt động

| # | Luồng | Dịch vụ |
|---|-------|---------|
| 1 | Truy cập domain, lọc traffic độc hại tại edge | Route 53, AWS WAF |
| 2 | Phục vụ giao diện React qua CDN + HTTPS | AWS Amplify Hosting |
| 3 | Đăng nhập, nhận JWT | Amazon Cognito |
| 4 | Xác thực token, route request | API Gateway (HTTP API) |
| 5 | Xử lý logic nghiệp vụ | AWS Lambda |
| 6 | Lưu trữ dữ liệu job | Amazon DynamoDB (+GSI) |
| 7 | Lưu trữ CV qua Presigned URL | Amazon S3 + AWS KMS |
| 8 | Nhắc nhở tự động hằng ngày | EventBridge + SES + SQS DLQ |
| 9 | Giám sát và cảnh báo admin | CloudWatch Alarm + SNS |
| 10 | Ghi log audit | AWS CloudTrail + S3 |

#### Nguyên tắc thiết kế

{{% notice info %}}
**Không dùng VPC** — kiến trúc thuần serverless, mọi thành phần là managed service được bảo vệ bằng IAM, WAF và KMS. Giảm độ phức tạp và loại bỏ chi phí NAT Gateway (~$32/tháng).

**Không dùng Scan trên DynamoDB** — mọi truy vấn đi theo partition key `userId` hoặc qua GSI, đảm bảo hiệu năng và chi phí thấp khi dữ liệu tăng.

**Cách ly dữ liệu tuyệt đối** — `userId` luôn lấy từ JWT đã được API Gateway xác thực, không bao giờ tin dữ liệu do client gửi lên.
{{% /notice %}}

Chi phí thực tế trong toàn bộ quá trình phát triển: **$0.85** (chi tiết ở mục 5.12).
