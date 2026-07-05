---
title: "Worklog Tuần 4"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.4. </b> "
---



### Mục tiêu tuần 4:

* Triển khai nhanh chóng các môi trường ứng dụng web bằng cách sử dụng dịch vụ tính toán đơn giản hóa và các tùy chọn container.
* Làm chủ kiến trúc mở rộng linh hoạt dựa trên lưu lượng truy cập và trạng thái sức khỏe của hạ tầng.
* Triển khai mạng phân phối nội dung toàn cầu để giảm thiểu độ trễ của ứng dụng.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | - Triển khai đơn giản với Amazon Lightsail: <br>&emsp; + Cài đặt nhanh ứng dụng WordPress hoặc môi trường Web trọn gói. <br>&emsp; + Quản lý ứng dụng container hóa thông qua Lightsail Containers.                                                                                        | 11/05/2026   | 11/05/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 3   | - Tự động mở rộng (EC2 Auto Scaling): <br>&emsp; + Xây dựng Launch Template để tự động hóa việc nhân bản máy chủ. <br>&emsp; + Cấu hình chính sách Scaling (Target Tracking) tự động theo lưu lượng truy cập.  <br>&emsp; + Kết hợp với Load Balancer để phân phối đều tải cho hệ thống.              | 12/05/2026   | 12/05/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 4   | - Tự động mở rộng (EC2 Auto Scaling): <br>&emsp; + Xây dựng Launch Template để tự động hóa việc nhân bản máy chủ. <br>&emsp; + Cấu hình chính sách Scaling (Target Tracking) tự động theo lưu lượng truy cập.  <br>&emsp; + Kết hợp với Load Balancer để phân phối đều tải cho hệ thống.                | 13/05/2026   | 13/05/2026      | <https://cloudjourney.awsstudygroup.com/> |> |
| 5   | - Mạng phân phối nội dung (Amazon CloudFront): <br>&emsp; + Tạo Distribution để tăng tốc độ tải trang web trên toàn thế giới. <br>&emsp; + Quản lý cơ chế Cache và các địa điểm biên (Edge Locations).               | 14/05/2026   | 14/05/2026      | <https://cloudjourney.awsstudygroup.com/> |> |
| 6   | - Mạng phân phối nội dung (Amazon CloudFront): <br>&emsp; + Tạo Distribution để tăng tốc độ tải trang web trên toàn thế giới. <br>&emsp; + Quản lý cơ chế Cache và các địa điểm biên (Edge Locations).              | 15/05/2026   | 15/05/2026      | <https://cloudjourney.awsstudygroup.com/> |> |


### Kết quả đạt được tuần 4:
* Tính toán đơn giản hóa với Amazon Lightsail:
  * Khởi tạo các cấu trúc mẫu cấu hình sẵn (Blueprints) như WordPress, LAMP, MEAN stack với mô hình giá cố định, dễ dự toán.
  * Quản lý các bản sao lưu hạ tầng (Snapshots), các quy tắc tường lửa và cấp phát IP tĩnh ngay trên bảng điều khiển Lightsail.

* Triển khai Container với Lightsail Containers:
  *	Đóng gói mã nguồn ứng dụng vào Docker Images và đẩy lên dịch vụ Lightsail Container Services.
  *	Cấu hình triển khai đa nút (multi-node), quản lý việc mở rộng container và giám sát trạng thái sức khỏe trực tiếp của container.

* Mở rộng quy mô linh hoạt với EC2 Auto Scaling:
  * Xây dựng các Launch Templates và Nhóm tự động mở rộng (Auto Scaling Groups - ASG) để tự động hóa tính co giãn của tài nguyên tính toán.
  *	Định nghĩa các chính sách mở rộng mục tiêu (Target Tracking) được kích hoạt bởi mức sử dụng CPU, lưu lượng mạng hoặc theo lịch trình tùy chỉnh.
  *	Tích hợp ASG với Bộ cân bằng tải ứng dụng (ALB) để định tuyến lưu lượng mượt mà dựa trên các kiểm tra sức khỏe (Health Checks) của mục tiêu.

* Mạng phân phối nội dung với Amazon CloudFront:
  * Tạo các CloudFront Distributions với nguồn từ S3 và ALB để tăng tốc độ phân phối tài nguyên trên toàn cầu.
  *	Cấu hình các chính sách bộ nhớ đệm, thiết lập thời gian tồn tại (TTL), chứng chỉ bảo mật SSL/TLS và giới hạn định tuyến theo địa lý.


