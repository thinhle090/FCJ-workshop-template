---
title: "Worklog Tuần 5"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.5. </b> "
---



### Mục tiêu tuần 5:

* Điều hướng lưu lượng internet hiệu quả bằng cách sử dụng các chính sách định tuyến tên miền đám mây nâng cao.
* Tích lũy kinh nghiệm thực hành toàn diện trong việc kết nối các cơ sở hạ tầng mạng hỗn hợp bảo mật.
* Thực thi các logic lập trình tại các vị trí biên toàn cầu nhằm nâng cao trải nghiệm người dùng.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | - Hệ thống tên miền (Amazon Route 53): <br>&emsp; + Đăng ký tên miền mới và quản lý các bản ghi DNS (A, CNAME, MX). <br>&emsp; + Thiết lập định tuyến thông minh: Failover (dự phòng) và Latency (giảm độ trễ).                                                                                        | 18/05/2026   | 18/05/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 3   | - Hội thảo thực hành Networking (AWS Workshop): <br>&emsp; + Thực hành tổng hợp về kiến trúc VPC nâng cao. <br>&emsp; + Kết nối mạng nội bộ với đám mây (Hybrid Cloud) một cách bảo mật.              | 19/05/2026   | 19/05/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 4   | - Hội thảo thực hành Networking (AWS Workshop): <br>&emsp; + Thực hành tổng hợp về kiến trúc VPC nâng cao. <br>&emsp; + Kết nối mạng nội bộ với đám mây (Hybrid Cloud) một cách bảo mật.               | 20/05/2026   | 20/05/2026      | <https://cloudjourney.awsstudygroup.com/> |> |
| 5   | - Xử lý logic tại biên (Lambda@Edge): <br>&emsp; + Triển khai mã nguồn xử lý trực tiếp tại các trạm biên của CloudFront. <br>&emsp; + Tối ưu hóa nội dung tùy biến dựa trên vị trí địa lý của người dùng.               | 21/05/2026   | 21/05/2026      | <https://cloudjourney.awsstudygroup.com/> |> |
| 6   | - Xử lý logic tại biên (Lambda@Edge): <br>&emsp; + Triển khai mã nguồn xử lý trực tiếp tại các trạm biên của CloudFront. <br>&emsp; + Tối ưu hóa nội dung tùy biến dựa trên vị trí địa lý của người dùng.               | 22/05/2026   | 22/05/2026      | <https://cloudjourney.awsstudygroup.com/> |> |


### Kết quả đạt được tuần 5:
* Quản lý hệ thống tên miền với Amazon Route 53:
  * Đăng ký tên miền internet và quản lý các loại bản ghi tiêu chuẩn bao gồm A, CNAME, Alias, MX và TXT.
  * Áp dụng các chính sách định tuyến nâng cao như: Weighted (Theo trọng số), Latency-based (Theo độ trễ), Failover (Dự phòng) và Geolocation (Theo vị trí địa lý).
  * Thiết lập các tính năng kiểm tra sức khỏe tự động (Health Checks) để ngay lập tức chuyển hướng lưu lượng người dùng khỏi các đầu cuối máy chủ bị lỗi.

* Thực hành mạng hỗn hợp (Hybrid Networking) qua AWS Workshop:
  *	Thực hiện các bài lab chuyên sâu để thiết lập kết nối bảo mật giữa hệ thống nội bộ (On-premises) và AWS VPC bằng Site-to-Site VPN.
  * Cấu hình các kết nối VPC Peering để cho phép định tuyến trực tiếp, bảo mật giữa các mạng ảo AWS bị cô lập với nhau.

* Tính toán máy chủ biên với Lambda@Edge:
  * Viết và triển khai các hàm Lambda (Node.js/Python) để can thiệp và sửa đổi các yêu cầu/phản hồi HTTP ngay tại các vị trí biên của CloudFront (Edge Locations).
  * Tối ưu hóa việc phân phối nội dung web một cách linh hoạt bằng cách xác thực người dùng và cá nhân hóa phương tiện truyền thông gần với vị trí địa lý của người dùng hơn.
