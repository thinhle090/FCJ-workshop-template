---
title: "Blog 3"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---

# [SECURITY/Architecture] TÁCH BẠCH DỮ LIỆU ĐỂ PHÂN TÍCH THỜI GIAN THỰC: BÀI HỌC TỪ OLDCASTLE VỚI AMAZON AURORA VÀ AMAZON QUICKSIGHT

### Giới thiệu

Trong quá trình nghiên cứu các kiến trúc phân tích dữ liệu thời gian thực trên Amazon Web Services (AWS), mình có tham khảo bài viết về mô hình triển khai hệ thống phân tích dữ liệu của **Oldcastle APG** trên AWS Architecture Blog. Bài viết tập trung vào cách xây dựng kiến trúc tách biệt giữa hệ thống giao dịch (OLTP) và hệ thống phân tích (Analytics), giúp khai thác dữ liệu theo thời gian thực mà không ảnh hưởng đến hiệu năng của hệ thống ERP.

Bài viết này tổng hợp và phân tích kiến trúc được AWS đề xuất, đồng thời làm rõ vai trò của từng thành phần trong việc xây dựng nền tảng phân tích dữ liệu thời gian thực theo hướng **Scalable Architecture**, đảm bảo khả năng mở rộng, tính sẵn sàng cao và hiệu quả khai thác dữ liệu.

### Bối cảnh

Trong các doanh nghiệp sản xuất và logistics, hệ thống ERP liên tục phát sinh các giao dịch mới như:

- Đơn hàng.
- Xuất nhập kho.
- Quản lý sản xuất.
- Quản lý khách hàng.

Các phòng ban luôn có nhu cầu theo dõi dữ liệu gần như theo thời gian thực để phục vụ quá trình ra quyết định.

Tuy nhiên, nếu các công cụ Business Intelligence (BI) truy vấn trực tiếp vào cơ sở dữ liệu giao dịch, hệ thống ERP có thể gặp các vấn đề như:

- Hiệu năng suy giảm.
- Gia tăng tải xử lý.
- Thời gian phản hồi kéo dài.
- Ảnh hưởng đến hoạt động nghiệp vụ.

Để giải quyết bài toán này, AWS giới thiệu kiến trúc phân tách hoàn toàn giữa hệ thống giao dịch và hệ thống phân tích thông qua cơ chế truyền dữ liệu theo thời gian thực (Real-time Data Streaming).

### 1. Thu thập dữ liệu theo thời gian thực

#### Vấn đề

Các thay đổi phát sinh trong hệ thống ERP cần được đồng bộ liên tục đến hệ thống phân tích mà không ảnh hưởng đến hoạt động của cơ sở dữ liệu giao dịch.

#### Phân tích

Oldcastle sử dụng **Infor Data Fabric Stream Pipelines** để thu thập các sự kiện phát sinh trong hệ thống ERP.

Các thay đổi như:

- Insert
- Update
- Delete

được truyền ngay lập tức sang hệ thống AWS thay vì chờ các tiến trình xử lý theo lô (Batch Processing).

#### Lợi ích

- Đồng bộ dữ liệu gần thời gian thực.
- Giảm độ trễ cập nhật dữ liệu.
- Không tạo tải truy vấn trực tiếp lên hệ thống ERP.


### 2. Kết nối an toàn vào môi trường AWS

#### Vấn đề

Hệ thống Infor Cloud ERP không thể truy cập trực tiếp vào cơ sở dữ liệu Amazon Aurora nằm trong Private Subnet.

Nếu mở truy cập trực tiếp sẽ làm tăng nguy cơ mất an toàn cho hạ tầng mạng.

#### Phân tích

AWS triển khai kiến trúc kết nối gồm:

- Amazon Route 53
- Network Load Balancer (NLB)
- EC2 Router (RDS Proxy Router)
- Amazon RDS Proxy

Network Load Balancer sử dụng Elastic IP cố định để tiếp nhận lưu lượng từ hệ thống ERP.

Các EC2 Router sử dụng cơ chế NAT để chuyển tiếp lưu lượng đến Amazon RDS Proxy, sau đó RDS Proxy quản lý toàn bộ kết nối tới Amazon Aurora PostgreSQL.

#### Lợi ích

- Bảo vệ cơ sở dữ liệu trong Private Subnet.
- Giảm số lượng kết nối trực tiếp đến Aurora.
- Hỗ trợ Connection Pooling.
- Tăng khả năng chịu tải và Failover.

### Hình 1. Kiến trúc phân tích dữ liệu thời gian thực của Oldcastle trên AWS

![Kiến trúc Real-time Analytics](/images/h3.1.jpg)

**Hình 1** minh họa kiến trúc triển khai hệ thống phân tích dữ liệu thời gian thực giữa Infor Cloud ERP và Amazon Web Services.

Luồng hoạt động gồm các bước chính:

1. Infor ERP phát sinh dữ liệu giao dịch và truyền các sự kiện thông qua Infor Data Fabric Stream Pipelines.
2. Dữ liệu được chuyển tới Amazon Route 53 và Network Load Balancer để định tuyến vào môi trường AWS.
3. EC2 Router tiếp nhận lưu lượng và chuyển tiếp thông qua Amazon RDS Proxy.
4. Amazon RDS Proxy quản lý kết nối đến Amazon Aurora PostgreSQL nhằm tối ưu hiệu năng và khả năng chịu tải.
5. Amazon Aurora lưu trữ dữ liệu giao dịch và hỗ trợ truy vấn dữ liệu bằng định dạng JSONB.
6. Amazon QuickSight kết nối đến Aurora để xây dựng Dashboard phân tích dữ liệu.
7. AWS Lambda phối hợp cùng Amazon API Gateway tạo URL truy cập động và xác thực người dùng.
8. Dashboard được nhúng trực tiếp vào giao diện Infor ERP thông qua Embedded Analytics, giúp người dùng truy cập báo cáo mà không cần chuyển sang hệ thống khác.

Kiến trúc này giúp tách biệt hoàn toàn tầng giao dịch và tầng phân tích, đồng thời đảm bảo dữ liệu luôn được cập nhật gần thời gian thực mà không ảnh hưởng đến hệ thống ERP.


### 3. Lưu trữ và xử lý dữ liệu

#### Vấn đề

Dữ liệu truyền theo thời gian thực thường có cấu trúc linh hoạt và thay đổi liên tục.

Việc lưu trữ theo mô hình quan hệ truyền thống có thể làm giảm tính linh hoạt khi phân tích.

#### Phân tích

Oldcastle sử dụng **Amazon Aurora PostgreSQL** với kiểu dữ liệu **JSONB** để lưu trữ dữ liệu streaming.

Các hàm JSON Native của PostgreSQL cho phép:

- Truy vấn linh hoạt.
- Phân tích dữ liệu bán cấu trúc.
- Dễ dàng mở rộng mô hình dữ liệu.

Đồng thời Aurora được triển khai theo kiến trúc **Multi-AZ** nhằm đảm bảo tính sẵn sàng cao.

#### Lợi ích

- Linh hoạt trong lưu trữ dữ liệu.
- Hỗ trợ truy vấn hiệu quả.
- Tăng khả năng mở rộng.
- Đảm bảo tính sẵn sàng của hệ thống.

### 4. Trực quan hóa dữ liệu với Amazon QuickSight

#### Phân tích

Amazon QuickSight được sử dụng để xây dựng các Dashboard phân tích dữ liệu thời gian thực.

Để tăng hiệu năng, hệ thống sử dụng:

- SPICE In-memory Engine.
- Embedded Dashboard.
- API Gateway.
- AWS Lambda.

Thông qua cơ chế Embedded Analytics, Dashboard được nhúng trực tiếp vào Infor ERP.

Người dùng không cần truy cập một hệ thống BI riêng biệt mà vẫn có thể xem báo cáo ngay trên giao diện làm việc quen thuộc.

#### Lợi ích

- Tăng tốc truy vấn Dashboard.
- Cải thiện trải nghiệm người dùng.
- Hỗ trợ phân quyền truy cập.
- Dễ dàng tích hợp với hệ thống ERP.


### Bảng tổng hợp kiến trúc

| Thành phần | Dịch vụ AWS | Vai trò |
|------------|-------------|---------|
| DNS | Amazon Route 53 | Định tuyến lưu lượng |
| Load Balancing | Network Load Balancer | Tiếp nhận kết nối từ ERP |
| Connection Management | Amazon RDS Proxy | Quản lý Connection Pool |
| Database | Amazon Aurora PostgreSQL | Lưu trữ dữ liệu giao dịch |
| Analytics | Amazon QuickSight | Phân tích và trực quan hóa dữ liệu |
| Integration | Amazon API Gateway | Cung cấp API tích hợp |
| Serverless | AWS Lambda | Xác thực và tạo Embedded URL |


### Kết luận

Kiến trúc được AWS giới thiệu cho thấy việc tách biệt giữa hệ thống giao dịch và hệ thống phân tích là một hướng tiếp cận hiệu quả đối với các doanh nghiệp có nhu cầu khai thác dữ liệu theo thời gian thực.

Việc kết hợp Infor Data Fabric Stream Pipelines với Amazon Aurora PostgreSQL, Amazon RDS Proxy và Amazon QuickSight giúp giảm tải cho hệ thống ERP, đồng thời cung cấp khả năng phân tích dữ liệu gần thời gian thực với hiệu năng và khả năng mở rộng cao.

Đây là một mô hình kiến trúc phù hợp cho các hệ thống ERP hiện đại, đặc biệt trong các lĩnh vực sản xuất, logistics và chuỗi cung ứng, nơi dữ liệu liên tục thay đổi và yêu cầu ra quyết định nhanh chóng.


### Tài liệu tham khảo

https://aws.amazon.com/.../real-time-analytics.../...

https://aws.amazon.com/.../real-time-analytics.../...

### Link bài viết

https://www.facebook.com/share/p/1DBmkQMGgv/