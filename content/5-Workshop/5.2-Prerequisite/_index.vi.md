---
title : "Chuẩn bị"
date : "2026-07-07"
weight : 2
chapter : false
pre : " <b> 5.2 </b> "
---

#### Nội dung
- [Tài khoản và công cụ](#tài-khoản-và-công-cụ)
- [Bước 1: Thiết lập Budget Alert](#bước-1-thiết-lập-budget-alert)
- [Bước 2: Chuẩn bị môi trường phát triển](#bước-2-chuẩn-bị-môi-trường-phát-triển)
- [Quy ước đặt tên](#quy-ước-đặt-tên)

#### Tài khoản và công cụ

| Yêu cầu | Ghi chú |
|---|---|
| Tài khoản AWS | Free Tier với promotional credits |
| IAM user | Tạo riêng cho từng thành viên, **không dùng root user** |
| Node.js 20 LTS | Chạy `node -v` để kiểm tra |
| Git + tài khoản GitHub | Dùng cho version control và deploy Amplify |
| Postman | Kiểm thử API |
| VS Code | Soạn thảo code |

#### Bước 1: Thiết lập Budget Alert

{{% notice warning %}}
Đây là việc **làm đầu tiên**, trước khi tạo bất kỳ tài nguyên nào. Budget alert là lớp bảo hiểm chống phát sinh chi phí ngoài ý muốn.
{{% /notice %}}

1. Đăng nhập AWS Console, vào **Billing and Cost Management**
2. Menu trái chọn **Budgets** → **Create budget**
3. Chọn **Zero spend budget** hoặc **Monthly cost budget** với ngưỡng **$5**
4. Nhập email của tất cả thành viên trong nhóm để nhận cảnh báo
5. Chọn **Create budget**

![Budget alert](/images/5-Workshop/5.2-Prerequisite/budget-alert.png)

#### Bước 2: Chuẩn bị môi trường phát triển

1. Cài **Node.js 20 LTS** từ [nodejs.org](https://nodejs.org), kiểm tra:
```bash
node -v    # v20.x hoặc v22.x
npm -v
```

2. Cài **Git** từ [git-scm.com](https://git-scm.com), kiểm tra: `git -v`

3. Tạo repository trên GitHub (đặt **Private** khi đang phát triển), mời các thành viên vào **Settings → Collaborators**

4. Thống nhất quy trình làm việc nhóm: **mỗi module một branch + Pull Request**
```bash
git checkout main && git pull
git checkout -b feat/ten-module
# ... code, commit ...
git push -u origin feat/ten-module
# Tạo Pull Request trên GitHub, đồng đội review rồi merge
```

#### Quy ước đặt tên

Thống nhất trước để tránh nhầm lẫn khi cấu hình IAM và biến môi trường:

| Tài nguyên | Tên |
|---|---|
| DynamoDB table | `JobTrackerJobs` |
| DynamoDB GSI | `FollowUpIndex` |
| Lambda | `job-tracker-crud-job`, `job-tracker-get-jobs`, `job-tracker-cv-presigned`, `lambda-followup-checker` |
| API Gateway | `job-tracker-api` |
| S3 bucket CV | `job-tracker-cv-<tên-nhóm>-2026` |
| SQS queue | `followup-dlq` |
| SNS topic | `job-tracker-alerts` |
| CloudTrail trail | `job-tracker-trail` |
| Amplify app | `job-tracker-web` |

{{% notice tip %}}
Tạo một file `NOTES.md` trong repo để ghi lại các giá trị quan trọng: User Pool ID, Client ID, API Invoke URL, tên bucket, Account ID. Cả nhóm cùng tra cứu.
{{% /notice %}}
