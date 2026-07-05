---
title: "Worklog Tuần 3"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.3. </b> "
---



### Mục tiêu tuần 3:

* Hiểu các khái niệm về lưu trữ đối tượng và triển khai lưu trữ trang web tĩnh.
* Nắm vững cách vận hành các dịch vụ cơ sở dữ liệu quản trị mã nguồn SQL và NoSQL trên AWS.
* Tìm hiểu cơ chế bộ nhớ đệm (caching) để tối ưu hóa hiệu năng và thời gian phản hồi của ứng dụng.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | - Dịch vụ lưu trữ đối tượng (Amazon S3): <br>&emsp; + Tạo Bucket, quản lý vòng đời tệp tin (Lifecycle) và phân quyền truy cập. <br>&emsp; + Triển khai lưu trữ trang web tĩnh (Static Website Hosting). <br>&emsp; + Cấu hình các chính sách bảo mật cho Bucket (Bucket Policy).                                                                                       | 04/05/2026   | 04/05/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 3   | - Cơ sở dữ liệu có cấu trúc (Amazon RDS): <br>&emsp; + Khởi tạo và quản lý MySQL/PostgreSQL trên đám mây. <br>&emsp; + Tìm hiểu cơ chế tự động sao lưu (Backup) và tính năng sẵn sàng cao.               | 05/05/2026   | 05/05/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 4   | - Cơ sở dữ liệu NoSQL (Amazon DynamoDB): <br>&emsp; + Hiểu về cấu trúc bảng (Table), Item và thuộc tính Primary Key. <br>&emsp; + Cách thức truy xuất dữ liệu với hiệu năng cực cao và khả năng mở rộng.               | 06/05/2026   | 06/05/2026      | <https://cloudjourney.awsstudygroup.com/> |> |
| 5   | - Cơ sở dữ liệu NoSQL (Amazon DynamoDB): <br>&emsp; + Hiểu về cấu trúc bảng (Table), Item và thuộc tính Primary Key. <br>&emsp; + Cách thức truy xuất dữ liệu với hiệu năng cực cao và khả năng mở rộng.               | 07/05/2026   | 07/05/2026      | <https://cloudjourney.awsstudygroup.com/> |> |
| 6   | - Tối ưu hiệu năng với Amazon ElastiCache: <br>&emsp; + Triển khai bộ nhớ đệm (Redis) để giảm tải cho cơ sở dữ liệu chính. <br>&emsp; + Phân tích các tình huống thực tế cần sử dụng Caching.               | 08/05/2026   | 08/05/2026      | <https://cloudjourney.awsstudygroup.com/> |> |


### Kết quả đạt được tuần 3:
* Dịch vụ lưu trữ đối tượng với Amazon S3:
  * Tạo và quản lý các S3 Bucket, thiết lập các quy tắc vòng đời tệp tin (Lifecycle) để tối ưu hóa chi phí lưu trữ.
  * Cấu hình tính năng Static Website Hosting và tích hợp thành công với các thiết lập tên miền riêng.
  * Kiểm soát bảo mật dữ liệu bằng cách định nghĩa chính xác các chính sách Bucket (Bucket Policy) và Danh sách kiểm soát truy cập (ACL).

* Dịch vụ cơ sở dữ liệu có cấu trúc với Amazon RDS:
  *	Triển khai các công cụ cơ sở dữ liệu quản trị phổ biến (MySQL, PostgreSQL) và cấu hình Multi-AZ để tăng tính sẵn sàng cao.
  *	Thiết lập cửa sổ sao lưu tự động (Backup), lịch trình bảo trì và tạo các bản sao chỉ đọc (Read Replicas) để giảm tải lưu lượng truy vấn.
  * Cô lập các thực thể cơ sở dữ liệu bên trong Subnet riêng tư (Private Subnet) và hạn chế truy cập mạng bằng các Security Group mục tiêu.

* Dịch vụ cơ sở dữ liệu NoSQL với Amazon DynamoDB:
  * Thiết kế các bảng cơ sở dữ liệu tối ưu bằng cách sử dụng Partition Key và Sort Key để hợp lý hóa việc truy vấn dữ liệu.
  *	Đánh giá giữa hai chế độ dung lượng Provisioned (Định sẵn) và On-demand (Theo nhu cầu) để cân bằng giữa hiệu suất và chi phí.
  *	Sử dụng DynamoDB Streams để theo dõi, ghi lại và phản hồi các thay đổi dữ liệu theo thời gian thực. 

* Bộ nhớ đệm trong với Amazon ElastiCache:
  * Triển khai các cụm (cluster) Redis và Memcached để tăng tốc độ truy xuất cho các truy vấn ứng dụng nặng.
  *	Phân tích các chiến lược bộ nhớ đệm cốt lõi như Lazy Loading và Write-through để duy trì tính nhất quán nghiêm ngặt của dữ liệu.


