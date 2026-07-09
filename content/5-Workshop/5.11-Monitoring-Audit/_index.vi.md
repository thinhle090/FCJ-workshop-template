---
title : "Giám sát và Audit"
date : "2026-07-07"
weight : 11
chapter : false
pre : " <b> 5.11 </b> "
---

#### Nội dung
- [Bước 1: Tạo SNS Topic](#bước-1-tạo-sns-topic)
- [Bước 2: Tạo CloudWatch Alarm](#bước-2-tạo-cloudwatch-alarm)
- [Bước 3: Bật CloudTrail](#bước-3-bật-cloudtrail)
- [Vai trò admin trong hệ thống](#vai-trò-admin-trong-hệ-thống)

#### Bước 1: Tạo SNS Topic

1. Vào **Amazon SNS** → **Topics** → **Create topic**
2. **Type**: **Standard**
3. **Name**: `job-tracker-alerts`
4. Giữ mặc định các mục còn lại → **Create topic**

Tạo subscription để nhận email:

1. Trong topic vừa tạo → **Create subscription**
2. **Protocol**: **Email**
3. **Endpoint**: email của admin
4. **Create subscription**
5. Mở hộp thư → bấm **Confirm subscription** trong email từ AWS
6. Refresh Console → trạng thái chuyển **Confirmed**

{{% notice warning %}}
Bước xác nhận subscription rất hay bị bỏ quên. Chưa **Confirmed** thì alarm có kêu cũng không gửi được email.
{{% /notice %}}

#### Bước 2: Tạo CloudWatch Alarm

Alarm cảnh báo khi API Gateway trả về quá nhiều lỗi 5xx.

1. Vào **CloudWatch** → **Alarms** → **All alarms** → **Create alarm**
2. **Select metric** → chọn namespace **ApiGateway** → chiều **ApiId**
3. Tick dòng có **ApiId** của bạn và **Metric name = 5xx** → **Select metric**
4. **Statistic**: **Sum**
5. **Period**: **5 minutes**
6. **Conditions**:
   - **Threshold type**: **Static**
   - **Whenever 5xx is**: **Greater/Equal**
   - **than**: `5`
7. **Next** → **Notification**:
   - **Alarm state trigger**: **In alarm**
   - **Send a notification to**: chọn topic `job-tracker-alerts`
8. **Next** → **Alarm name**: `api-5xx-alarm` → **Next** → **Create alarm**

![CloudWatch alarm](/images/5-Workshop/5.11-Monitoring-Audit/cloudwatch-alarm.png)

{{% notice info %}}
Ngưỡng "5 lỗi 5xx trong 5 phút" đủ nhạy để phát hiện sự cố thật, đủ trễ để không cảnh báo oan vì một lỗi lẻ. Alarm mới tạo sẽ ở trạng thái **Insufficient data**, sau vài phút chuyển **OK**.
{{% /notice %}}

Nếu metric **5xx** không xuất hiện trong danh sách: metric chỉ sinh ra sau khi API có traffic. Gọi vài request rồi refresh lại.

#### Bước 3: Bật CloudTrail

CloudTrail ghi lại **mọi API call trong account** — không chỉ API Gateway mà cả thao tác với Lambda, DynamoDB, S3, Cognito, IAM.

1. Vào **AWS CloudTrail** → **Trails** → **Create trail**
2. **Trail name**: `job-tracker-trail`
3. **Storage location**: chọn **Create new S3 bucket** (tên tự sinh dạng `aws-cloudtrail-logs-...`)
4. **Log file SSE-KMS encryption**: bỏ tick (bucket đã có SSE-S3 mặc định)
5. **CloudWatch Logs**: bỏ qua
6. **Next** → **Event type**: giữ **Management events**, tick cả **Read** và **Write**
7. **Next** → **Create trail**

Đợi vài phút, vào **Event history** để xem danh sách API call gần đây.

![CloudTrail logging](/images/5-Workshop/5.11-Monitoring-Audit/cloudtrail-logging.png)

Kiểm tra bucket S3 → thấy cấu trúc `AWSLogs/<ACCOUNT_ID>/CloudTrail/ap-southeast-1/...` chứa các file log `.json.gz`.

{{% notice tip %}}
Khi có sự cố, Event history cho phép truy vết **ai đã làm gì, lúc nào, từ IP nào**. Đây là công cụ audit cơ bản nhưng thiết yếu cho mọi hệ thống production.
{{% /notice %}}

#### Vai trò admin trong hệ thống

Ứng dụng **chủ đích không có role admin trong app**: dữ liệu ứng tuyển và CV là thông tin nhạy cảm, nguyên tắc least privilege áp dụng cho cả người vận hành. Không có API nào cho phép một người dùng đọc dữ liệu của người khác.

Quản trị hệ thống thực hiện ở **tầng hạ tầng**:
- **IAM** — phân quyền truy cập AWS resources
- **CloudWatch Alarm + SNS** — cảnh báo sự cố qua email
- **CloudTrail** — truy vết mọi thao tác

Hướng mở rộng nếu cần thống kê tổng hợp: dùng **Cognito Groups** — thêm user vào group `admin`, JWT sẽ có claim `cognito:groups`, Lambda kiểm tra claim này để mở các API riêng (ví dụ đếm tổng số user, tổng số job) mà không đọc nội dung chi tiết.
