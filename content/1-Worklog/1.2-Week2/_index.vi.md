---
title: "Worklog Tuần 2"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2:

* Hiểu về billing để tránh mất tiền khi thực hành.
* Nắm và hiểu được User, Role, Policy.
* Làm quen thao tác thực tế khi thực hành.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | - Kiến thức mạng cơ bản với Amazon Virtual Private Cloud (VPC): <br>&emsp; + Thiết lập sơ đồ mạng với Subnet (Public/Private) và IP Addressing. <br>&emsp; + Cấu hình Route Table, Internet Gateway để quản lý luồng dữ liệu. <br>&emsp; + Cách phân biệt và sử dụng Network ACLs so với Security Groups.                                                                                       | 27/04/2026   | 27/04/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 3   | - Kiến thức máy chủ cơ bản (EC2): <br>&emsp; + Khởi tạo Instance, lựa chọn cấu hình CPU/RAM phù hợp với nhu cầu <br>&emsp; + Quản lý bảo mật qua Key Pair và các quy tắc Inbound/Outbound.               | 28/04/2026   | 28/04/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 4   | - Kiến thức máy chủ cơ bản (EC2): <br>&emsp; + Khởi tạo Instance, lựa chọn cấu hình CPU/RAM phù hợp với nhu cầu <br>&emsp; + Quản lý bảo mật qua Key Pair và các quy tắc Inbound/Outbound.               | 29/04/2026   | 29/04/2026      | <https://cloudjourney.awsstudygroup.com/> |> |
| 5   | - Phân quyền với IAM Roles: <br>&emsp; + Tạo Role cho EC2 để phân quyền thực thi an toàn. <br>&emsp; + Gán chính sách (Policy) truy cập các dịch vụ nội bộ như S3 hoặc CloudWatch.               | 30/04/2026   | 30/04/2026      | <https://cloudjourney.awsstudygroup.com/> |> |
| 6   | - Giám sát với CloudWatch: <br>&emsp; + Theo dõi các thông số kỹ thuật thực tế như CPU Utilization, Network In/Out. <br>&emsp; + Xem nhật ký hoạt động (Logs) của hệ thống. <br>&emsp; + Thiết lập Alarms để tự động gởi cảnh báo qua Email khi có sự cố.               | 01/05/2026   | 01/05/2026      | <https://cloudjourney.awsstudygroup.com/> |> |


### Kết quả đạt được tuần 2:
* Kiến thức mạng cơ bản với Amazon VPC:
  * Thiết lập sơ đồ mạng tùy chỉnh với các dải IP CIDR phù hợp.
  * Phân biệt và cấu hình Subnet công khai (Public) cho Web Server và Subnet riêng tư (Private) cho Database.
  * Thiết lập bảng định tuyến (Route Table), Internet Gateway và NAT Gateway để quản lý luồng dữ liệu ra vào. 
  * Sử dụng Network ACL và Security Group để tạo ra nhiều lớp bảo mật cho hệ thống mạng.

* Kiến thức máy chủ cơ bản với Amazon EC2:
  *	Lựa chọn Family Instance (T, M, C, R) dựa trên nhu cầu tối ưu hóa CPU hay RAM cho ứng dụng.
  *	Thực hành khởi tạo, quản lý vòng đời Instance và sử dụng User Data để tự động hóa cài đặt khi khởi động.
  * Quản lý lưu trữ với EBS (Elastic Block Store) và hiểu về các loại Volume (gp2, gp3, io1).

* Phân quyền thực thi với IAM Roles cho EC2:
  * Khởi tạo IAM Roles và gán các chính sách (Policies) bảo mật theo nguyên tắc quyền hạn tối thiểu.
  *	Gán Role cho EC2 để truy cập an toàn vào S3, DynamoDB mà không cần lưu trữ Access Key trên máy chủ.
  *	Kiểm tra và gỡ lỗi quyền truy cập thông qua IAM Policy Simulator. 

* Giám sát hệ thống với Amazon CloudWatch:
  * Theo dõi các chỉ số cơ bản (CPU, Disk, Network) và cài đặt Unified CloudWatch Agent để lấy chỉ số RAM.
  *	Thiết lập CloudWatch Alarms để tự động gởi thông báo qua SNS khi tài nguyên vượt ngưỡng.
  *	Phân tích log hệ thống và log ứng dụng thông qua CloudWatch Logs Insights.


