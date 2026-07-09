---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# [SECURITY/Architecture] Đừng để Cloud thành điểm mù: Tư duy hạ tầng "miễn nhiễm" với Ransomware trên AWS

### Giới thiệu

Trong quá trình nghiên cứu về kiến trúc bảo mật trên nền tảng Amazon Web Services (AWS), mình có tham khảo bài viết **"Let's Architect! Architecting for Security"** từ AWS Architecture Blog. Nội dung bài viết không tập trung giới thiệu từng dịch vụ riêng lẻ mà phân tích cách xây dựng kiến trúc hệ thống theo hướng **Security by Design**, giúp giảm bề mặt tấn công (Attack Surface) và nâng cao khả năng phòng thủ nhiều lớp (Defense in Depth).

Bài viết này tổng hợp và phân tích các nguyên tắc kỹ thuật nổi bật được đề cập trong tài liệu, đồng thời làm rõ vai trò của từng thành phần trong việc giảm thiểu rủi ro từ các cuộc tấn công Ransomware trên môi trường AWS.

### Bối cảnh

Các cuộc tấn công Ransomware hiện đại thường không bắt đầu bằng việc mã hóa dữ liệu ngay lập tức. Thay vào đó, kẻ tấn công sẽ trải qua nhiều giai đoạn nhằm chiếm quyền kiểm soát hệ thống trước khi thực hiện hành vi phá hoại.

Chuỗi tấn công điển hình bao gồm:

1. Thu thập thông tin (Reconnaissance)
2. Dò quét mạng (Network Scanning)
3. Leo thang đặc quyền (Privilege Escalation)
4. Di chuyển ngang (Lateral Movement)
5. Đánh cắp dữ liệu (Data Exfiltration)
6. Mã hóa dữ liệu (Data Encryption)

Nếu hệ thống được xây dựng với kiến trúc mạng và mô hình phân quyền chưa hợp lý, kẻ tấn công có thể nhanh chóng mở rộng phạm vi xâm nhập và làm gia tăng mức độ ảnh hưởng của sự cố.

Theo quan điểm của AWS, việc thiết kế kiến trúc bảo mật ngay từ đầu có thể giúp ngăn chặn hoặc làm gián đoạn chuỗi tấn công này trước khi thiệt hại nghiêm trọng xảy ra.


### 1. Quản lý đặc quyền bằng Temporary Elevated Access

#### Vấn đề

Trong nhiều hệ thống, các tài khoản quản trị hoặc IAM Role được cấp quyền đặc biệt trong thời gian dài. Khi thông tin xác thực bị rò rỉ, kẻ tấn công có thể sử dụng ngay các quyền này để kiểm soát hệ thống.

#### Phân tích

AWS khuyến nghị áp dụng mô hình **Temporary Elevated Access (Just-In-Time Access)**.

Theo mô hình này:

- Quyền quản trị chỉ được cấp khi thực sự cần thiết.
- Quyền truy cập có thời gian hiệu lực giới hạn.
- Sau khi hết thời gian quy định, quyền sẽ tự động bị thu hồi.

#### Lợi ích

- Giảm nguy cơ lộ quyền quản trị.
- Hạn chế khả năng leo thang đặc quyền.
- Tuân thủ nguyên tắc Least Privilege.
- Thu hẹp cơ hội khai thác của kẻ tấn công.

### 2. Cô lập lưu lượng mạng bằng Amazon VPC Endpoints

#### Vấn đề

Một số hệ thống đặt EC2 hoặc Lambda trong Private Subnet nhưng vẫn sử dụng NAT Gateway để truy cập các dịch vụ AWS như Amazon S3 hoặc Amazon DynamoDB.

Mặc dù tài nguyên nằm trong mạng riêng, lưu lượng vẫn có thể đi qua Internet trước khi quay lại hệ thống AWS.

Điều này làm tăng nguy cơ:

- Thiết lập kết nối Command and Control (C&C).
- Rò rỉ dữ liệu ra bên ngoài.
- Tải thêm mã độc từ Internet.

#### Phân tích

AWS khuyến nghị sử dụng **VPC Gateway Endpoint** hoặc **Interface Endpoint** để kết nối trực tiếp đến các dịch vụ AWS.

Khi đó, toàn bộ lưu lượng sẽ được truyền trong mạng nội bộ của AWS mà không cần đi qua Internet.

#### Hình 1. Kiến trúc Temporary Elevated Access

![Kiến trúc Temporary Elevated Access](/images/h1.jpg)

**Hình 1** minh họa quy trình cấp quyền tạm thời (Temporary Elevated Access). Thay vì cấp quyền quản trị lâu dài cho người dùng, hệ thống sử dụng một **Temporary Elevated Access Broker** làm trung gian quản lý quyền truy cập.

Quy trình hoạt động gồm bốn bước:

1. Người dùng được xác thực thông qua **Identity Provider (IdP)**.
2. Broker kiểm tra danh tính, vai trò và quyền được yêu cầu.
3. Broker có thể tích hợp với hệ thống quản lý thay đổi (Change Management) hoặc quản lý sự cố (Incident Management) để xác thực yêu cầu trước khi cấp quyền.
4. Sau khi được phê duyệt, Broker cấp quyền truy cập tạm thời vào môi trường AWS trong khoảng thời gian giới hạn, sau đó quyền sẽ tự động bị thu hồi.

Kiến trúc này giúp giảm thiểu nguy cơ lộ quyền quản trị, hạn chế khả năng leo thang đặc quyền và tuân thủ nguyên tắc **Least Privilege**.

#### Lợi ích

- Giảm nguy cơ lộ quyền quản trị.
- Hạn chế khả năng leo thang đặc quyền.
- Tuân thủ nguyên tắc Least Privilege.
- Thu hẹp cơ hội khai thác của kẻ tấn công.

### 2. Cô lập lưu lượng mạng bằng Amazon VPC Endpoints

#### Vấn đề

Một số hệ thống đặt EC2 hoặc Lambda trong Private Subnet nhưng vẫn sử dụng NAT Gateway để truy cập các dịch vụ AWS như Amazon S3 hoặc Amazon DynamoDB.

Mặc dù tài nguyên nằm trong mạng riêng, lưu lượng vẫn có thể đi qua Internet trước khi quay lại hệ thống AWS.

Điều này làm tăng nguy cơ:

- Thiết lập kết nối Command and Control (C&C).
- Rò rỉ dữ liệu ra bên ngoài.
- Tải thêm mã độc từ Internet.

#### Phân tích

AWS khuyến nghị sử dụng **VPC Gateway Endpoint** hoặc **Interface Endpoint** để kết nối trực tiếp đến các dịch vụ AWS.

Khi đó, toàn bộ lưu lượng sẽ được truyền trong mạng nội bộ của AWS mà không cần đi qua Internet.

#### Lợi ích

- Giảm lưu lượng Internet không cần thiết.
- Hạn chế nguy cơ rò rỉ dữ liệu.
- Giảm khả năng thiết lập kết nối điều khiển từ xa.
- Tăng cường tính bảo mật cho hạ tầng mạng.

#### Giám sát

Khi kết hợp với **VPC Flow Logs**, hệ thống có thể phát hiện các hoạt động bất thường như:

- Quét cổng (Port Scanning)
- Dò quét mạng (Network Scanning)
- Brute Force SSH
- Kết nối đến địa chỉ IP không xác định


### 3. Tích hợp Shift Left Security vào quy trình CI/CD

#### Vấn đề

Nếu hoạt động kiểm tra bảo mật chỉ được thực hiện sau khi triển khai, chi phí khắc phục sẽ cao và nguy cơ đưa lỗ hổng vào môi trường Production sẽ tăng lên.

#### Phân tích

Shift Left Security là phương pháp đưa các hoạt động kiểm tra bảo mật vào ngay từ giai đoạn phát triển phần mềm.

Các kỹ thuật phổ biến gồm:

- Phân tích mã nguồn tĩnh (SAST)
- Quét thông tin bí mật (Secret Scanning)
- Quét thư viện phụ thuộc (Dependency Scanning)
- Kiểm tra Infrastructure as Code (IaC Scanning)
- Quét Container Image

#### Lợi ích

Có thể phát hiện sớm:

- Hardcoded Credentials
- SQL Injection
- Cross-Site Scripting (XSS)
- Thư viện có lỗ hổng
- Lỗi cấu hình hạ tầng

Qua đó giảm thiểu rủi ro trước khi ứng dụng được triển khai lên môi trường thực tế.


### 4. Giám sát bảo mật tập trung với AWS Security Reference Architecture (SRA)

#### Vấn đề

Đối với các doanh nghiệp vận hành nhiều tài khoản AWS, dữ liệu giám sát thường phân tán, gây khó khăn trong việc phát hiện và xử lý sự cố.

#### Phân tích

AWS Security Reference Architecture (SRA) đề xuất xây dựng một tài khoản bảo mật chuyên biệt để tập trung toàn bộ dữ liệu giám sát.

Các dịch vụ thường được tích hợp gồm:

- Amazon GuardDuty
- AWS Security Hub
- Amazon Macie
- AWS CloudTrail
- AWS Config

#### Hình 2. AWS Security Reference Architecture (SRA)

![AWS Security Reference Architecture](/images/h2.jpg)

**Hình 2** mô tả mô hình tham chiếu bảo mật của AWS (AWS Security Reference Architecture – SRA). Kiến trúc được tổ chức theo nhiều Organizational Unit (OU), mỗi OU đảm nhiệm một chức năng riêng trong việc quản lý và bảo vệ hệ thống.

Các thành phần chính gồm:

- **Organization Management**: Quản lý toàn bộ tài khoản AWS, CloudTrail, IAM Identity Center và AWS Config.
- **OU – Security**: Tập trung các dịch vụ giám sát như Amazon GuardDuty, AWS Security Hub, Amazon Macie, AWS Config và AWS Lambda để tự động phản ứng với sự cố.
- **OU – Infrastructure**: Triển khai các dịch vụ hạ tầng mạng như AWS WAF, AWS Shield Advanced, AWS Network Firewall, Route 53 và CloudFront nhằm bảo vệ lớp mạng và ứng dụng.
- **OU – Workloads**: Chứa các tài nguyên triển khai ứng dụng như Amazon EC2, Amazon Aurora, Amazon S3, AWS Secrets Manager, AWS KMS và Amazon Cognito.

Mô hình này giúp chuẩn hóa việc quản lý bảo mật trên nhiều tài khoản AWS, đồng thời tập trung toàn bộ dữ liệu giám sát về một trung tâm điều hành bảo mật (Security Account), giúp rút ngắn thời gian phát hiện và xử lý sự cố.

#### Lợi ích

Kiến trúc này giúp:

- Quản lý tập trung.
- Theo dõi toàn bộ môi trường AWS.
- Tương quan sự kiện bảo mật.
- Giảm thời gian phát hiện sự cố (MTTD).
- Giảm thời gian phản ứng sự cố (MTTR).

#### Bảng tổng hợp các nguyên tắc bảo mật

| Nguyên tắc | Dịch vụ/Khái niệm AWS | Mục tiêu |
|------------|------------------------|----------|
| Temporary Elevated Access | IAM Role | Hạn chế leo thang đặc quyền |
| Kết nối nội bộ | Amazon VPC Endpoints | Loại bỏ truy cập Internet không cần thiết |
| Shift Left Security | CI/CD Security Pipeline | Phát hiện lỗ hổng từ sớm |
| Giám sát tập trung | AWS Security Reference Architecture (SRA) | Nâng cao khả năng phát hiện và ứng phó sự cố |

### Kết luận

Bảo mật trên nền tảng AWS không chỉ phụ thuộc vào việc triển khai các dịch vụ bảo mật mà còn phụ thuộc vào cách thiết kế kiến trúc hệ thống ngay từ đầu.

Các nguyên tắc như quản lý quyền truy cập tạm thời, cô lập lưu lượng mạng, tích hợp kiểm tra bảo mật trong quy trình phát triển và xây dựng hệ thống giám sát tập trung đều góp phần giảm bề mặt tấn công, hạn chế khả năng di chuyển ngang của kẻ tấn công và nâng cao hiệu quả phát hiện cũng như ứng phó sự cố.

Việc áp dụng các nguyên tắc này theo tư duy **Security by Design** sẽ giúp xây dựng môi trường AWS an toàn hơn, tăng khả năng chống chịu trước các cuộc tấn công Ransomware và các mối đe dọa an ninh mạng hiện đại.


### Tài liệu tham khảo

1. https://aws.amazon.com/.../lets-architect.../...
2. https://aws.amazon.com/.../lets-architect.../...

### Link bài viết: 
1. https://www.facebook.com/groups/awsstudygroupfcj/permalink/2195119847919642/?rdid=0ofV8lHxkOypQfWq#