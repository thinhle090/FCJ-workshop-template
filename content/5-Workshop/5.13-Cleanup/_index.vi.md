---
title : "Dọn dẹp tài nguyên"
date : "2026-07-07"
weight : 13
chapter : false
pre : " <b> 5.13 </b> "
---

Sau khi hoàn thành workshop, dọn dẹp tài nguyên theo thứ tự sau để tránh phát sinh chi phí:

#### Ưu tiên cao (có phí định kỳ)

1. **AWS WAF trên Amplify** (~$23/tháng)
   - Amplify → app → **Hosting → Firewall** → **Actions** → **Remove firewall**

2. **Route 53 Hosted Zone** ($0.50/tháng)
   - Route 53 → **Hosted zones** → chọn zone → xoá các record đã tạo (trừ NS và SOA) → **Delete zone**

3. **EventBridge Scheduler** (dừng cron trước tiên)
   - EventBridge → **Scheduler → Schedules** → chọn `daily-followup-check` → **Delete**

#### Các tài nguyên còn lại

4. **Amplify app**
   - Amplify → app → **App settings → General** → **Delete app**

5. **API Gateway**
   - API Gateway → chọn `job-tracker-api` → **Delete**

6. **Lambda functions** (4 functions)
   - Lambda → chọn từng function → **Actions → Delete**
   - IAM → **Roles** → xoá các role `job-tracker-*-role-*` và `lambda-followup-checker-role-*`

7. **DynamoDB**
   - DynamoDB → bảng `JobTrackerJobs` → tắt **Deletion protection** trước → **Delete table**

8. **S3 buckets** (bucket phải rỗng mới xoá được)
   - Bucket CV: **Empty** → **Delete**
   - Bucket CloudTrail logs: **Empty** → **Delete**

9. **CloudTrail**
   - CloudTrail → **Trails** → chọn `job-tracker-trail` → **Delete**

10. **CloudWatch**
    - **Alarms** → xoá `api-5xx-alarm`
    - **Log groups** → xoá các log group `/aws/lambda/...`

11. **SNS / SQS**
    - SNS → xoá topic `job-tracker-alerts`
    - SQS → xoá queue `followup-dlq`

12. **Cognito**
    - Cognito → chọn user pool → **Delete user pool**

13. **SES**
    - SES → **Identities** → xoá các email đã verify

#### Nên giữ lại

- **Budget alert** — miễn phí và hữu ích cho các dự án sau
- **Repository GitHub** — lưu trữ code và lịch sử làm việc

{{% notice tip %}}
Sau 1–2 ngày, kiểm tra lại **Billing → Bills**: mọi dòng chi phí về $0 nghĩa là đã dọn sạch. Nếu còn dòng nào phát sinh, dùng **Cost Explorer** để tìm dịch vụ còn sót.
{{% /notice %}}
