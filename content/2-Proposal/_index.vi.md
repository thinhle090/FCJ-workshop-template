---
title: "Bản đề xuất"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Job Tracker Platform using AWS Serverless Architecture
## Nền tảng quản lý quá trình ứng tuyển việc làm trên kiến trúc AWS Serverless. 

### 1. Tóm tắt điều hành  
Job Tracker Platform là nền tảng hỗ trợ sinh viên và người tìm việc quản lý toàn bộ quá trình ứng tuyển trên một hệ thống tập trung, bao gồm quản lý công việc, lưu trữ CV, theo dõi trạng thái ứng tuyển và tự động gửi email nhắc nhở follow-up.

Hệ thống được xây dựng trên kiến trúc AWS Serverless, sử dụng AWS Amplify Hosting, Amazon Cognito, Amazon API Gateway, AWS Lambda, Amazon DynamoDB và Amazon S3 nhằm đảm bảo khả năng mở rộng, bảo mật và tối ưu chi phí vận hành. Bên cạnh đó, hệ thống tích hợp AWS WAF, AWS KMS, Amazon EventBridge Scheduler, Amazon SES, Amazon CloudWatch và AWS CloudTrail để tăng cường bảo mật, tự động hóa và giám sát hoạt động, đồng thời tạo nền tảng cho việc mở rộng các tính năng phân tích dữ liệu và AI trong tương lai.

### 2. Tuyên bố vấn đề  
*Vấn đề hiện tại*  
Hiện nay, nhiều sinh viên và người tìm việc vẫn quản lý quá trình ứng tuyển bằng các công cụ thủ công như Microsoft Excel, Google Sheets hoặc ghi chú trên điện thoại. Mỗi hồ sơ ứng tuyển thường bao gồm nhiều thông tin như tên công ty, vị trí tuyển dụng, trạng thái xử lý, lịch phỏng vấn, thời gian cần gửi email follow-up và các tài liệu đính kèm như CV hoặc Cover Letter. Khi số lượng đơn ứng tuyển tăng lên, việc quản lý bằng các phương pháp này trở nên khó khăn, dễ xảy ra sai sót và mất nhiều thời gian.

Bên cạnh đó, các tệp CV và tài liệu thường được lưu trữ ở nhiều vị trí khác nhau trên máy tính hoặc các dịch vụ lưu trữ đám mây, khiến người dùng khó tìm kiếm và quản lý tập trung. Việc không có cơ chế phân quyền và bảo vệ dữ liệu phù hợp cũng làm tăng nguy cơ truy cập trái phép hoặc làm lộ thông tin cá nhân.

Ngoài ra, người dùng thường phải tự ghi nhớ thời điểm cần liên hệ lại với nhà tuyển dụng sau khi nộp hồ sơ. Nếu bỏ lỡ các mốc thời gian quan trọng như lịch phỏng vấn hoặc thời điểm follow-up, cơ hội việc làm có thể bị ảnh hưởng. Các nền tảng quản lý tuyển dụng hiện có chủ yếu hướng đến doanh nghiệp hoặc yêu cầu trả phí để sử dụng đầy đủ tính năng, trong khi chưa có nhiều giải pháp phù hợp cho sinh viên và người tìm việc cá nhân.

Về mặt kỹ thuật, nhiều ứng dụng web truyền thống yêu cầu triển khai và quản lý máy chủ, cơ sở dữ liệu cùng hạ tầng mạng, dẫn đến chi phí vận hành cao, khó mở rộng khi số lượng người dùng tăng và mất nhiều công sức trong việc giám sát, bảo trì cũng như đảm bảo an toàn hệ thống.

*Giải pháp*  
Để giải quyết các vấn đề trên, đề tài xây dựng Job Tracker Platform dựa trên kiến trúc AWS Serverless, giúp quản lý toàn bộ quá trình ứng tuyển trên một nền tảng tập trung, an toàn và có khả năng mở rộng linh hoạt.

Giao diện người dùng được phát triển bằng React và triển khai trên AWS Amplify Hosting, cho phép tự động triển khai (CI/CD) từ GitHub và phân phối nội dung thông qua mạng CDN toàn cầu với chứng chỉ HTTPS mặc định. Người dùng truy cập hệ thống thông qua tên miền riêng được quản lý bởi Amazon Route 53, trong khi AWS WAF bảo vệ ứng dụng trước các cuộc tấn công phổ biến như SQL Injection, Cross-Site Scripting (XSS), bot độc hại và các địa chỉ IP có nguy cơ cao.

Hệ thống sử dụng Amazon Cognito để quản lý đăng ký, đăng nhập và xác thực người dùng bằng JWT Token. Sau khi xác thực thành công, các yêu cầu được gửi đến Amazon API Gateway, nơi JWT Authorizer kiểm tra tính hợp lệ của token trước khi chuyển tiếp đến các hàm AWS Lambda xử lý nghiệp vụ. Các chức năng chính bao gồm quản lý công việc ứng tuyển, theo dõi trạng thái hồ sơ, lưu trữ thông tin tuyển dụng và tạo Presigned URL phục vụ việc tải lên hoặc tải xuống tệp CV.

Thông tin ứng tuyển được lưu trữ trên Amazon DynamoDB, trong đó mọi truy vấn dữ liệu đều dựa trên userId lấy từ JWT nhằm đảm bảo mỗi người dùng chỉ có thể truy cập dữ liệu của chính mình. Các tệp CV và tài liệu đính kèm được lưu trên Amazon S3 Private Bucket, người dùng chỉ truy cập thông qua Presigned URL có thời gian hiệu lực giới hạn, giúp tăng cường bảo mật và giảm tải cho hệ thống xử lý.

Để hỗ trợ người dùng không bỏ lỡ các mốc quan trọng trong quá trình tuyển dụng, Amazon EventBridge Scheduler được cấu hình chạy theo lịch hằng ngày nhằm kích hoạt Lambda kiểm tra các hồ sơ cần follow-up. Khi đến thời điểm quy định, hệ thống tự động gửi email nhắc nhở thông qua Amazon SES và cập nhật trạng thái nhằm tránh gửi trùng lặp. Trong trường hợp xảy ra lỗi, cơ chế Retry và Amazon SQS Dead Letter Queue (DLQ) giúp lưu giữ các tác vụ chưa xử lý thành công để phục vụ việc khắc phục sau này.

Ngoài ra, hệ thống sử dụng Amazon CloudWatch để giám sát log, metric và thiết lập cảnh báo khi phát hiện lỗi, đồng thời AWS CloudTrail ghi nhận toàn bộ lịch sử thao tác trên tài khoản AWS nhằm hỗ trợ kiểm toán và điều tra khi có sự cố bảo mật.

*Lợi ích và hoàn vốn đầu tư (ROI)*  
Việc triển khai Job Tracker Platform trên nền tảng AWS Serverless mang lại nhiều lợi ích cả về kỹ thuật lẫn chi phí vận hành. Nhờ mô hình Pay-as-you-go, hệ thống chỉ phát sinh chi phí khi tài nguyên thực sự được sử dụng, đồng thời không cần đầu tư hoặc quản lý máy chủ vật lý. Trong môi trường thử nghiệm và dự án học tập, phần lớn các dịch vụ như AWS Lambda, Amazon API Gateway, Amazon DynamoDB, Amazon S3, Amazon Cognito, Amazon SES, Amazon SNS, Amazon SQS, Amazon CloudWatch và AWS CloudTrail đều nằm trong AWS Free Tier, giúp giảm đáng kể chi phí triển khai. Chi phí phát sinh chủ yếu đến từ AWS WAF và Amazon Route 53, với tổng chi phí ước tính khoảng 24 USD/tháng, đồng thời vẫn được bù đắp bởi AWS Free Tier Credits, nên chi phí thực tế trong quá trình phát triển gần như bằng 0 USD.

Đối với người dùng, hệ thống giúp quản lý tập trung toàn bộ quá trình ứng tuyển trên một giao diện duy nhất, hỗ trợ lưu trữ hồ sơ, theo dõi trạng thái ứng tuyển và tự động gửi email nhắc nhở follow-up. Điều này giúp giảm thời gian quản lý thủ công, hạn chế bỏ lỡ các cơ hội việc làm và nâng cao hiệu quả tìm kiếm việc làm.

Về mặt kỹ thuật, kiến trúc Serverless cho phép hệ thống tự động mở rộng theo số lượng người dùng mà không cần thay đổi hạ tầng. Đồng thời, việc kết hợp Amazon Cognito, AWS WAF, Presigned URL, Amazon CloudWatch và AWS CloudTrail giúp tăng cường bảo mật, khả năng giám sát và kiểm toán hoạt động của hệ thống. Kiến trúc này cũng tạo nền tảng thuận lợi để mở rộng các tính năng trong tương lai như phân tích dữ liệu ứng tuyển, thống kê hiệu quả tìm việc, tích hợp trí tuệ nhân tạo (AI) và các dịch vụ Machine Learning mà không cần thay đổi kiến trúc tổng thể.

### 3. Kiến trúc giải pháp  
#### Mục tiêu kiến trúc

Kiến trúc của **Job Tracker Platform** được xây dựng theo mô hình **AWS Serverless Architecture** nhằm phát triển một hệ thống quản lý quá trình ứng tuyển việc làm có khả năng mở rộng linh hoạt, đảm bảo tính sẵn sàng cao, tăng cường bảo mật và tối ưu chi phí vận hành. Toàn bộ hệ thống được triển khai trên các dịch vụ được quản lý (Managed Services) của AWS, giúp loại bỏ nhu cầu quản lý máy chủ và tự động mở rộng theo lưu lượng truy cập thực tế.

Kiến trúc được thiết kế theo hướng tách biệt giữa các lớp giao diện người dùng, xử lý nghiệp vụ, lưu trữ dữ liệu và giám sát hệ thống. Đồng thời, các cơ chế xác thực, phân quyền, mã hóa dữ liệu và giám sát hoạt động được tích hợp ngay từ đầu theo nguyên tắc **Security by Design**, góp phần đảm bảo tính bảo mật và độ tin cậy của toàn bộ hệ thống.

#### Kiến trúc cụ thể

Hệ thống được xây dựng theo mô hình nhiều lớp (Multi-tier Architecture), bao gồm các thành phần chính sau:

- **Lớp truy cập (Edge Layer):** Người dùng truy cập hệ thống thông qua tên miền được quản lý bởi **Amazon Route 53**. Mọi yêu cầu đều được kiểm tra bởi **AWS WAF** nhằm ngăn chặn các cuộc tấn công phổ biến như SQL Injection (SQLi), Cross-site Scripting (XSS), bot độc hại và các địa chỉ IP có mức độ rủi ro cao trước khi chuyển đến ứng dụng.

- **Lớp giao diện (Frontend):** Giao diện người dùng được phát triển bằng **React** và triển khai trên **AWS Amplify Hosting**. Amplify tự động triển khai ứng dụng từ GitHub thông qua quy trình CI/CD, đồng thời phân phối nội dung thông qua mạng CDN toàn cầu với HTTPS được cấu hình tự động.

- **Lớp xác thực (Authentication):** Người dùng đăng nhập thông qua **Amazon Cognito**. Sau khi xác thực thành công, Cognito cấp **JSON Web Token (JWT)** để xác minh danh tính và quyền truy cập trong các yêu cầu tiếp theo.

- **Lớp xử lý nghiệp vụ (Backend):** Các yêu cầu từ ứng dụng được gửi đến **Amazon API Gateway (HTTP API)**. JWT Authorizer xác thực token trước khi chuyển yêu cầu đến các hàm **AWS Lambda** tương ứng để xử lý các chức năng như quản lý công việc, truy vấn dữ liệu và tạo Presigned URL.

- **Lớp dữ liệu (Data Layer):** Thông tin ứng tuyển được lưu trữ trên **Amazon DynamoDB**, trong khi các tệp CV và tài liệu được lưu trong **Amazon S3 Private Bucket**. Toàn bộ dữ liệu được mã hóa bằng **AWS KMS** nhằm đảm bảo tính bảo mật.

- **Lớp tự động hóa (Automation):** **Amazon EventBridge Scheduler** kích hoạt Lambda theo lịch để kiểm tra các hồ sơ đến thời điểm cần follow-up. Lambda gửi email nhắc nhở thông qua **Amazon SES**, cập nhật trạng thái đã gửi và sử dụng **Amazon SQS Dead Letter Queue (DLQ)** để lưu các yêu cầu gửi thất bại sau nhiều lần thử lại.

- **Lớp giám sát và kiểm toán (Monitoring & Auditing):** **Amazon CloudWatch** thu thập nhật ký và chỉ số hoạt động của hệ thống, đồng thời gửi cảnh báo qua **Amazon SNS** khi phát hiện lỗi. **AWS CloudTrail** ghi nhận toàn bộ API Call và lưu nhật ký vào một S3 Bucket riêng phục vụ mục đích kiểm toán.

![Job Tracker Architecture](/images/sdt2.png)


#### Dịch vụ AWS sử dụng

| Dịch vụ AWS | Vai trò |
|-------------|----------|
| **Amazon Route 53** | Quản lý DNS và định tuyến người dùng đến hệ thống |
| **AWS WAF** | Bảo vệ ứng dụng trước SQL Injection, XSS, bot và các truy cập độc hại |
| **AWS Amplify Hosting** | Triển khai và phân phối ứng dụng React, hỗ trợ CI/CD |
| **Amazon Cognito** | Đăng ký, đăng nhập và xác thực người dùng bằng JWT |
| **Amazon API Gateway (HTTP API)** | Tiếp nhận và định tuyến các yêu cầu API |
| **AWS Lambda** | Xử lý toàn bộ nghiệp vụ của hệ thống |
| **Amazon DynamoDB** | Lưu trữ dữ liệu ứng tuyển và thông tin người dùng |
| **Amazon S3** | Lưu trữ giao diện tĩnh và các tệp CV dưới dạng Private Bucket |
| **AWS KMS** | Mã hóa dữ liệu lưu trữ trên DynamoDB và Amazon S3 |
| **Amazon EventBridge Scheduler** | Tự động kích hoạt các tác vụ theo lịch |
| **Amazon SES** | Gửi email nhắc nhở follow-up |
| **Amazon SQS (DLQ)** | Lưu các tác vụ gửi email thất bại để xử lý lại |
| **Amazon CloudWatch** | Giám sát hoạt động, thu thập log và metric |
| **Amazon SNS** | Gửi cảnh báo đến quản trị viên |
| **AWS CloudTrail** | Ghi nhận lịch sử hoạt động và phục vụ kiểm toán |

#### Thiết kế thành phần

- **Lớp biên (Edge Layer):** Bao gồm **Amazon Route 53**, **AWS WAF** và **AWS Amplify Hosting**, chịu trách nhiệm định tuyến người dùng, bảo vệ ứng dụng khỏi các cuộc tấn công phổ biến và phân phối giao diện React thông qua CDN toàn cầu.

- **Lớp giao diện người dùng (Frontend Layer):** Ứng dụng React được triển khai trên AWS Amplify, cho phép người dùng quản lý quá trình ứng tuyển, cập nhật trạng thái công việc, tải lên hoặc tải xuống CV và theo dõi lịch follow-up thông qua trình duyệt web.

- **Lớp xác thực (Authentication Layer):** **Amazon Cognito** thực hiện đăng ký, đăng nhập và cấp JWT Token. API Gateway sử dụng JWT Authorizer để xác thực người dùng trước khi cho phép truy cập các API.

- **Lớp xử lý nghiệp vụ (Application Layer):** **AWS Lambda** đảm nhiệm toàn bộ nghiệp vụ như quản lý công việc, truy xuất dữ liệu, tạo Presigned URL cho S3 và xử lý gửi email nhắc nhở theo lịch.

- **Lớp lưu trữ dữ liệu (Data Layer):** **Amazon DynamoDB** lưu trữ thông tin người dùng và dữ liệu ứng tuyển, trong khi **Amazon S3 Private Bucket** lưu các tệp CV và tài liệu. Toàn bộ dữ liệu được mã hóa bằng AWS KMS để đảm bảo an toàn.

- **Lớp tự động hóa (Automation Layer):** **Amazon EventBridge Scheduler** kích hoạt Lambda theo lịch để kiểm tra các hồ sơ cần follow-up. Lambda gửi email qua Amazon SES và sử dụng Amazon SQS Dead Letter Queue để lưu các yêu cầu gửi thất bại nhằm tránh mất dữ liệu.

- **Lớp giám sát và kiểm toán (Monitoring & Auditing Layer):** **Amazon CloudWatch** thu thập log, metric và tạo cảnh báo thông qua Amazon SNS khi phát hiện lỗi hệ thống. **AWS CloudTrail** ghi lại toàn bộ hoạt động của tài khoản AWS và lưu trữ nhật ký phục vụ kiểm toán và điều tra sự cố.

### 4. Triển khai kỹ thuật  
*Các giai đoạn triển khai*  
#### Triển khai hạ tầng

Hệ thống được triển khai trên nền tảng **AWS Serverless** nhằm giảm chi phí vận hành và loại bỏ việc quản lý máy chủ. Tên miền được cấu hình thông qua **Amazon Route 53**, giao diện người dùng được triển khai trên **AWS Amplify Hosting**, trong khi **AWS WAF** được tích hợp để bảo vệ ứng dụng trước các cuộc tấn công phổ biến như SQL Injection (SQLi), Cross-site Scripting (XSS) và các truy cập độc hại từ Internet.

#### Triển khai ứng dụng

Giao diện người dùng được phát triển bằng **React** và quản lý mã nguồn trên GitHub. AWS Amplify tự động xây dựng (Build) và triển khai (Deploy) ứng dụng mỗi khi có thay đổi trên kho mã nguồn. Người dùng đăng nhập thông qua **Amazon Cognito**, nhận **JWT Token** và sử dụng token này để truy cập các API được cung cấp bởi **Amazon API Gateway**.

#### Triển khai Backend

Các chức năng nghiệp vụ được xây dựng dưới dạng các hàm **AWS Lambda** và được định tuyến thông qua **Amazon API Gateway (HTTP API)**. JWT Authorizer xác thực người dùng trước khi chuyển yêu cầu đến Lambda tương ứng để xử lý các chức năng như quản lý công việc, truy vấn dữ liệu và tạo Presigned URL phục vụ việc tải lên hoặc tải xuống tệp CV.

#### Triển khai dữ liệu

Thông tin về quá trình ứng tuyển được lưu trữ trên **Amazon DynamoDB**, trong khi các tệp CV được lưu trong **Amazon S3 Private Bucket**. Các thao tác tải lên và tải xuống tệp sử dụng **Presigned URL** nhằm cho phép người dùng truy cập trực tiếp vào S3 mà không cần truyền dữ liệu qua Lambda. Dữ liệu trên DynamoDB và S3 đều được mã hóa bằng **AWS KMS** nhằm đảm bảo an toàn thông tin.

#### Triển khai tự động hóa

Hệ thống sử dụng **Amazon EventBridge Scheduler** để tự động kích hoạt Lambda theo lịch trình kiểm tra các hồ sơ đến thời điểm cần follow-up. Lambda gửi email nhắc nhở thông qua **Amazon SES**, sau đó cập nhật trạng thái để tránh gửi lặp lại. Nếu việc gửi email thất bại sau nhiều lần thử, thông điệp sẽ được chuyển đến **Amazon SQS Dead Letter Queue (DLQ)** để xử lý sau.

#### Giám sát và bảo mật

Toàn bộ hoạt động của hệ thống được theo dõi thông qua **Amazon CloudWatch**, bao gồm nhật ký (Logs), chỉ số hoạt động (Metrics) và cảnh báo (Alarms). Khi số lượng lỗi vượt ngưỡng cấu hình, CloudWatch sẽ gửi thông báo đến quản trị viên thông qua **Amazon SNS**. Đồng thời, **AWS CloudTrail** ghi nhận toàn bộ API Call phát sinh trong tài khoản AWS và lưu trữ nhật ký trên Amazon S3 nhằm phục vụ công tác kiểm toán và điều tra sự cố.

* Tóm tắt công nghệ sử dụng

| Thành phần | Công nghệ / Dịch vụ |
|------------|---------------------|
| Frontend | React, HTML, CSS, JavaScript |
| Source Control | Git, GitHub |
| Hosting | AWS Amplify Hosting |
| Domain & DNS | Amazon Route 53 |
| Authentication | Amazon Cognito (JWT) |
| API | Amazon API Gateway (HTTP API) |
| Business Logic | AWS Lambda |
| Database | Amazon DynamoDB |
| File Storage | Amazon S3 (Private Bucket) |
| File Security | Amazon S3 Presigned URL |
| Encryption | AWS Key Management Service (AWS KMS) |
| Scheduler | Amazon EventBridge Scheduler |
| Email Service | Amazon Simple Email Service (SES) |
| Queue | Amazon SQS (Dead Letter Queue) |
| Notification | Amazon Simple Notification Service (SNS) |
| Monitoring | Amazon CloudWatch |
| Audit Logging | AWS CloudTrail |
| Security | AWS WAF |


* Quy trình triển khai
1. Phát triển giao diện người dùng bằng React và quản lý mã nguồn trên GitHub.
2. Triển khai ứng dụng lên AWS Amplify Hosting và cấu hình tên miền bằng Amazon Route 53.
3. Tích hợp AWS WAF để bảo vệ ứng dụng trước các cuộc tấn công từ Internet.
4. Cấu hình Amazon Cognito để quản lý người dùng và xác thực bằng JWT Token.
5. Xây dựng các API trên Amazon API Gateway (HTTP API).
6. Phát triển các hàm AWS Lambda xử lý nghiệp vụ của hệ thống.
7. Lưu trữ dữ liệu ứng tuyển trên Amazon DynamoDB và các tệp CV trên Amazon S3 Private Bucket.
8. Sử dụng Presigned URL để người dùng tải lên và tải xuống tệp trực tiếp từ Amazon S3.
9. Thiết lập Amazon EventBridge Scheduler để tự động kiểm tra các hồ sơ cần follow-up và gửi email thông qua Amazon SES.
10. Cấu hình Amazon SQS Dead Letter Queue để lưu các yêu cầu gửi email thất bại.
11. Thiết lập Amazon CloudWatch, Amazon SNS và AWS CloudTrail để giám sát, gửi cảnh báo và ghi nhận nhật ký hoạt động của hệ thống.

*Yêu cầu kỹ thuật*  
| Hạng mục | Yêu cầu kỹ thuật |
|:----------|:-----------------|
| **Nền tảng điện toán đám mây** | Amazon Web Services (AWS) |
| **Frontend** | React, HTML5, CSS3, JavaScript |
| **Quản lý mã nguồn** | Git, GitHub |
| **Hosting** | AWS Amplify Hosting |
| **Tên miền & DNS** | Amazon Route 53 |
| **Xác thực người dùng** | Amazon Cognito (JWT Authentication) |
| **API Backend** | Amazon API Gateway (HTTP API) |
| **Xử lý nghiệp vụ** | AWS Lambda |
| **Cơ sở dữ liệu** | Amazon DynamoDB |
| **Lưu trữ tệp** | Amazon S3 (Private Bucket, Presigned URL) |
| **Mã hóa dữ liệu** | AWS Key Management Service (AWS KMS) |
| **Lập lịch tác vụ** | Amazon EventBridge Scheduler |
| **Gửi Email** | Amazon Simple Email Service (SES) |
| **Hàng đợi xử lý lỗi** | Amazon SQS Dead Letter Queue |
| **Giám sát hệ thống** | Amazon CloudWatch |
| **Thông báo cảnh báo** | Amazon Simple Notification Service (SNS) |
| **Kiểm toán hệ thống** | AWS CloudTrail |
| **Bảo mật ứng dụng** | AWS WAF |
| **Môi trường phát triển** | Visual Studio Code, Node.js, npm |

### 5. Lộ trình & Mốc triển khai

- **Giai đoạn chuẩn bị (Tháng 0):**
    - Khảo sát nhu cầu của người dùng và phân tích bài toán quản lý quá trình ứng tuyển.
    - Nghiên cứu các dịch vụ AWS Serverless phù hợp với yêu cầu của hệ thống.
    - Thiết kế kiến trúc tổng thể, mô hình dữ liệu và luồng hoạt động của hệ thống.

- **Giai đoạn phát triển (Tháng 1):**
    - Xây dựng giao diện người dùng bằng React.
    - Triển khai ứng dụng trên AWS Amplify Hosting và cấu hình tên miền với Amazon Route 53.
    - Thiết lập Amazon Cognito để quản lý người dùng và xác thực bằng JWT.

- **Giai đoạn xây dựng Backend (Tháng 2):**
    - Xây dựng các API bằng Amazon API Gateway và AWS Lambda.
    - Thiết kế cơ sở dữ liệu trên Amazon DynamoDB.
    - Triển khai lưu trữ tệp CV trên Amazon S3 bằng Presigned URL.
    - Tích hợp AWS KMS để mã hóa dữ liệu.

- **Giai đoạn hoàn thiện hệ thống (Tháng 3):**
    - Tích hợp Amazon EventBridge Scheduler và Amazon SES để gửi email nhắc nhở tự động.
    - Thiết lập Amazon CloudWatch, Amazon SNS và AWS CloudTrail để giám sát và ghi nhật ký.
    - Cấu hình AWS WAF nhằm tăng cường bảo mật cho ứng dụng.
    - Kiểm thử chức năng, tối ưu hiệu năng và triển khai phiên bản hoàn chỉnh.

- **Giai đoạn vận hành và mở rộng:**
    - Theo dõi hoạt động của hệ thống thông qua CloudWatch.
    - Tối ưu chi phí và hiệu năng dựa trên lưu lượng sử dụng thực tế.
    - Phát triển thêm các tính năng như thống kê quá trình ứng tuyển, phân tích dữ liệu và tích hợp AI hỗ trợ người dùng trong tương lai.
### 6. Ước tính ngân sách  
Có thể xem chi phí trên [AWS Pricing Calculator](https://calculator.aws/#/estimate?id=621f38b12a1ef026842ba2ddfe46ff936ed4ab01)  
Hoặc tải [tệp ước tính ngân sách](../attachments/budget_estimation.pdf).  

*Chi phí hạ tầng*  
| Dịch vụ AWS | Mục đích sử dụng | Chi phí ước tính (USD/tháng) |
|:------------|:-----------------|-----------------------------:|
| AWS Amplify Hosting | Lưu trữ và triển khai ứng dụng React | $0.00 *(Free Tier)* |
| Amazon Route 53 Hosted Zone | Quản lý DNS | $0.50 |
| AWS WAF | Bảo vệ ứng dụng Web | ~$23.00 |
| Amazon Cognito | Xác thực người dùng | $0.00 *(Free Tier)* |
| Amazon API Gateway (HTTP API) | Quản lý API | Đã bao gồm |
| AWS Lambda | Xử lý nghiệp vụ | Đã bao gồm |
| Amazon DynamoDB | Lưu trữ dữ liệu | Đã bao gồm |
| Amazon S3 | Lưu trữ tệp CV | Đã bao gồm |
| AWS KMS | Mã hóa dữ liệu | Đã bao gồm |
| Amazon EventBridge Scheduler | Lập lịch tác vụ | Đã bao gồm |
| Amazon SES | Gửi email | Đã bao gồm |
| Amazon SNS | Gửi cảnh báo | Đã bao gồm |
| Amazon SQS | Dead Letter Queue | Đã bao gồm |
| Amazon CloudWatch | Giám sát hệ thống | Đã bao gồm |
| AWS CloudTrail | Ghi nhật ký hoạt động | Đã bao gồm |

### Tổng chi phí

| Hạng mục | Chi phí |
|----------|---------:|
| Backend AWS (Lambda, API Gateway, DynamoDB, S3, Cognito, SES, SNS, SQS, CloudWatch, CloudTrail, KMS) | **~$0.85/tháng** |
| AWS Amplify Hosting | **$0.00** *(Free Tier)* |
| Amazon Route 53 Hosted Zone | **$0.50/tháng** |
| AWS WAF | **~$23.00/tháng** |
| **Tổng chi phí ước tính** | **~$24.35/tháng** |

*Tổng*: 24.35 USD/tháng, 292,2 USD/12 tháng  
- *Lưu ý*: Đây là chi phí ước tính dành cho môi trường demo hoặc dự án học tập. Nếu lượng người dùng và lưu lượng truy cập tăng, các dịch vụ như CloudFront, Lambda, API Gateway, DynamoDB và S3 sẽ tăng chi phí theo mức sử dụng thực tế. AWS áp dụng mô hình Pay-as-you-go, tức chỉ thanh toán cho tài nguyên đã sử dụng. 

### 7. Đánh giá rủi ro  
#### Rủi ro về bảo mật

Hệ thống có nguy cơ bị truy cập trái phép nếu cơ chế xác thực hoặc phân quyền được cấu hình không chính xác. Các API có thể trở thành mục tiêu của những cuộc tấn công như SQL Injection, Cross-site Scripting (XSS), Bot hoặc khai thác lỗ hổng xác thực. Ngoài ra, dữ liệu CV và thông tin ứng tuyển là những dữ liệu nhạy cảm cần được bảo vệ khỏi nguy cơ rò rỉ.

Để giảm thiểu rủi ro, hệ thống sử dụng Amazon Cognito để xác thực người dùng bằng JWT, API Gateway JWT Authorizer để kiểm tra token trước khi chuyển yêu cầu đến Lambda, AWS WAF để chặn các cuộc tấn công phổ biến ở tầng Web, đồng thời sử dụng S3 Private Bucket và AWS KMS để mã hóa dữ liệu lưu trữ.


#### Rủi ro về hiệu năng

Khi số lượng người dùng hoặc số lượng yêu cầu tăng đột biến, hệ thống có thể phát sinh độ trễ trong quá trình xử lý hoặc gia tăng số lượng lỗi nếu tài nguyên không được mở rộng phù hợp.

Kiến trúc Serverless trên AWS giúp giảm thiểu rủi ro này thông qua khả năng tự động mở rộng của AWS Lambda, Amazon API Gateway và Amazon DynamoDB. Ngoài ra, AWS Amplify Hosting kết hợp với CDN toàn cầu giúp giảm độ trễ truy cập đối với người dùng ở nhiều khu vực khác nhau.

#### Rủi ro về lưu trữ dữ liệu

Dữ liệu ứng tuyển và các tệp CV là tài sản quan trọng của người dùng. Nếu cấu hình lưu trữ hoặc phân quyền không hợp lý, dữ liệu có thể bị truy cập trái phép hoặc mất tính toàn vẹn.

Để đảm bảo an toàn, toàn bộ CV được lưu trữ trong Amazon S3 Private Bucket và chỉ được truy cập thông qua Presigned URL có thời hạn ngắn. Dữ liệu trong Amazon DynamoDB và Amazon S3 đều được mã hóa bằng AWS KMS nhằm đảm bảo tính bảo mật trong quá trình lưu trữ.


#### Rủi ro về chi phí

Mặc dù kiến trúc Serverless giúp tối ưu chi phí, việc số lượng người dùng hoặc lưu lượng truy cập tăng cao vẫn có thể làm phát sinh chi phí lớn hơn dự kiến, đặc biệt đối với các dịch vụ như AWS WAF, API Gateway hoặc Lambda.

Để kiểm soát ngân sách, hệ thống sử dụng Amazon CloudWatch để theo dõi mức sử dụng tài nguyên, đồng thời tận dụng AWS Free Tier và Billing Dashboard nhằm giám sát chi phí và tối ưu tài nguyên trong quá trình vận hành.


#### Rủi ro trong quá trình vận hành

Trong quá trình triển khai hoặc cập nhật hệ thống có thể phát sinh lỗi ứng dụng, lỗi cấu hình hoặc sự cố dịch vụ, ảnh hưởng đến trải nghiệm của người dùng.

Để nâng cao khả năng vận hành, hệ thống sử dụng Amazon CloudWatch Logs và CloudWatch Alarms để theo dõi lỗi theo thời gian thực. Khi số lượng lỗi API vượt ngưỡng cấu hình, CloudWatch sẽ gửi cảnh báo thông qua Amazon SNS đến quản trị viên. Ngoài ra, EventBridge Scheduler được cấu hình Retry và SQS Dead Letter Queue (DLQ) để xử lý các tác vụ gửi email thất bại, trong khi AWS CloudTrail ghi nhận toàn bộ hoạt động API phục vụ kiểm tra và khắc phục sự cố.

### 8. Kết quả kỳ vọng  
#### Hoàn thành hệ thống

Xây dựng thành công Job Tracker Platform trên kiến trúc AWS Serverless, đáp ứng đầy đủ các chức năng như quản lý quá trình ứng tuyển, lưu trữ CV, theo dõi trạng thái hồ sơ, tải lên và tải xuống tài liệu an toàn, đồng thời hỗ trợ gửi email nhắc nhở tự động.


#### Nâng cao trải nghiệm người dùng

Người dùng có thể quản lý toàn bộ quá trình ứng tuyển trên một nền tảng duy nhất với giao diện trực quan. Hệ thống hỗ trợ cập nhật trạng thái công việc, lưu trữ hồ sơ tập trung và tự động gửi email nhắc lịch follow-up, giúp hạn chế bỏ lỡ các mốc quan trọng trong quá trình tuyển dụng.


#### Đảm bảo hiệu năng và bảo mật

Hệ thống vận hành trên kiến trúc Serverless có khả năng tự động mở rộng theo nhu cầu sử dụng, đồng thời áp dụng nhiều lớp bảo vệ như AWS WAF, Amazon Cognito, JWT Authorizer, S3 Private Bucket, AWS KMS và CloudTrail nhằm đảm bảo tính bảo mật, tính toàn vẹn dữ liệu và khả năng giám sát hệ thống.


#### Tăng khả năng vận hành và giám sát

Toàn bộ hoạt động của hệ thống được theo dõi thông qua Amazon CloudWatch, CloudTrail và SNS Alert, giúp phát hiện sớm sự cố, gửi cảnh báo đến quản trị viên và hỗ trợ quá trình kiểm tra, khắc phục lỗi một cách nhanh chóng.


#### Khả năng mở rộng trong tương lai

Kiến trúc được thiết kế theo hướng tách biệt giữa giao diện, tầng xử lý nghiệp vụ và tầng dữ liệu, giúp dễ dàng mở rộng các chức năng mới như thống kê quá trình ứng tuyển, phân tích dữ liệu, tích hợp AI hỗ trợ tối ưu CV, gợi ý việc làm hoặc xây dựng hệ thống phân tích hành vi người dùng mà không cần thay đổi kiến trúc tổng thể.