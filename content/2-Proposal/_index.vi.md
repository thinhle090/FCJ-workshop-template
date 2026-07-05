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
Job Tracker Platform là nền tảng tập trung giúp sinh viên và người tìm việc tối ưu hóa quá trình ứng tuyển thay thế Excel thủ công. Hệ thống hỗ trợ quản lý CV, theo dõi trạng thái hồ sơ và tự động gửi email nhắc nhở follow-up nhà tuyển dụng.

Hệ thống được triển khai trên kiến trúc AWS Serverless (React, S3, CloudFront, Cognito, API Gateway, Lambda, DynamoDB) giúp tự động mở rộng và tối ưu chi phí vận hành. Hệ thống còn tích hợp EventBridge, SES để tự động hóa email và WAF, CloudWatch để bảo mật, tạo nền tảng vững chắc để phát triển các tính năng phân tích dữ liệu và AI trong tương lai.

### 2. Tuyên bố vấn đề  
*Vấn đề hiện tại*  
Hiện nay, nhiều sinh viên và người tìm việc vẫn quản lý quá trình ứng tuyển bằng các phương pháp thủ công như sử dụng Microsoft Excel, Google Sheets hoặc ghi chú trên điện thoại. Việc lưu trữ thông tin phân tán khiến người dùng khó theo dõi trạng thái của từng hồ sơ, dễ bỏ sót các mốc thời gian quan trọng như lịch phỏng vấn, thời hạn phản hồi hoặc thời điểm cần liên hệ lại với nhà tuyển dụng.

Bên cạnh đó, các tệp hồ sơ như CV, thư xin việc và các tài liệu liên quan thường được lưu trữ ở nhiều vị trí khác nhau trên máy tính hoặc các dịch vụ lưu trữ đám mây, gây mất thời gian khi tìm kiếm và quản lý. Người dùng cũng không có công cụ để thống kê số lượng công việc đã ứng tuyển, đánh giá hiệu quả tìm việc hoặc theo dõi tiến độ ứng tuyển theo từng giai đoạn.

Ngoài ra, nhiều hệ thống quản lý ứng tuyển hiện có chỉ cung cấp các chức năng cơ bản hoặc yêu cầu trả phí để sử dụng đầy đủ tính năng. Điều này khiến sinh viên và người mới tìm việc khó tiếp cận một giải pháp quản lý toàn diện với chi phí hợp lý. Vì vậy, cần có một nền tảng tập trung, an toàn và dễ sử dụng, giúp quản lý toàn bộ quá trình ứng tuyển, đồng thời tận dụng các dịch vụ AWS Serverless để giảm chi phí vận hành, tăng khả năng mở rộng và tự động hóa các tác vụ như gửi email nhắc nhở và quản lý hồ sơ.

*Giải pháp*  
Giải pháp sử dụng Amazon Cognito để xác thực và quản lý người dùng, đảm bảo chỉ những người đã đăng nhập mới có quyền truy cập vào hệ thống. Giao diện web được phát triển bằng React và triển khai trên Amazon S3, kết hợp với Amazon CloudFront và Amazon Route 53 để phân phối nội dung nhanh chóng và quản lý tên miền. Các chức năng nghiệp vụ được xử lý thông qua Amazon API Gateway và AWS Lambda, giúp giảm chi phí vận hành và không cần quản lý máy chủ.

Dữ liệu về quá trình ứng tuyển được lưu trữ trên Amazon DynamoDB, trong khi các tệp CV và tài liệu đính kèm được lưu trên Amazon S3 dưới dạng lưu trữ riêng tư (Private Bucket). Hệ thống sử dụng Amazon EventBridge để tự động kiểm tra các hồ sơ đến thời điểm cần follow-up và kích hoạt AWS Lambda gửi email nhắc nhở thông qua Amazon SES, giúp người dùng không bỏ lỡ các mốc thời gian quan trọng trong quá trình tuyển dụng.

Ngoài ra, giải pháp còn tích hợp AWS WAF để bảo vệ ứng dụng trước các truy cập độc hại, Amazon CloudWatch để giám sát hoạt động và hiệu năng hệ thống, cùng AWS CloudTrail để ghi nhận lịch sử thao tác trên môi trường AWS. Với kiến trúc Serverless, hệ thống có khả năng mở rộng linh hoạt theo số lượng người dùng, tối ưu chi phí và tạo nền tảng để phát triển thêm các tính năng như thống kê quá trình ứng tuyển, phân tích dữ liệu và tích hợp AI trong tương lai.

*Lợi ích và hoàn vốn đầu tư (ROI)*  
Việc triển khai Job Tracker Platform trên nền tảng AWS Serverless giúp giảm chi phí đầu tư và vận hành nhờ mô hình thanh toán theo mức sử dụng thực tế (Pay-as-you-go), đồng thời không cần quản lý máy chủ. Hệ thống giúp người dùng quản lý tập trung quá trình ứng tuyển, giảm thời gian theo dõi hồ sơ và hạn chế bỏ lỡ các mốc quan trọng nhờ tính năng nhắc lịch tự động. Bên cạnh đó, kiến trúc Serverless có khả năng mở rộng linh hoạt, tạo nền tảng để phát triển thêm các tính năng mới trong tương lai, từ đó mang lại hiệu quả sử dụng lâu dài và tối ưu giá trị đầu tư.

### 3. Kiến trúc giải pháp  
*Mục tiêu kiến trúc*  
Kiến trúc của Job Tracker Platform được thiết kế theo mô hình AWS Serverless nhằm xây dựng một hệ thống có khả năng mở rộng linh hoạt, đảm bảo tính bảo mật, giảm chi phí vận hành và không yêu cầu quản lý máy chủ. Hệ thống được xây dựng theo hướng tách biệt giữa giao diện người dùng, tầng xử lý nghiệp vụ và tầng lưu trữ dữ liệu, giúp dễ dàng bảo trì, nâng cấp và phát triển các tính năng mới trong tương lai.


*Kiến trúc cụ thể* 
* Kiến trúc hệ thống bao gồm ba lớp chính:
* Lớp giao diện (Frontend): Được phát triển bằng React và triển khai trên Amazon S3, kết hợp với Amazon CloudFront và Route 53 để cung cấp nội dung nhanh chóng đến người dùng.
* Lớp xử lý nghiệp vụ (Backend): Người dùng đăng nhập thông qua Amazon Cognito. Các yêu cầu từ giao diện được gửi đến Amazon API Gateway và được xử lý bởi các hàm AWS Lambda để thực hiện các chức năng của hệ thống.
* Lớp lưu trữ dữ liệu (Data Layer): Dữ liệu ứng tuyển được lưu trữ trên Amazon DynamoDB, trong khi các tệp CV và tài liệu được lưu trên Amazon S3. Amazon EventBridge và Amazon SES được sử dụng để tự động gửi email nhắc nhở, còn CloudWatch và CloudTrail hỗ trợ giám sát và theo dõi hoạt động của hệ thống.

![Job Tracker Architecture](/images/sd.png)


*Dịch vụ AWS sử dụng*  
- *Amazon Route 53*: Quản lý tên miền và định tuyến người dùng đến hệ thống.
- *AWS WAF*: Bảo vệ ứng dụng trước các truy cập và tấn công độc hại.
- *Amazon CloudFront*: Phân phối nội dung với tốc độ cao và giảm độ trễ truy cập.
- *Amazon S3*: Lưu trữ giao diện React và các tệp CV của người dùng.
- *Amazon Cognito*: Xác thực người dùng và quản lý quyền truy cập. 
- *Amazon API Gateway*: Tiếp nhận và định tuyến các yêu cầu API đến Lambda.  
- *AWS Lambda*: Xử lý các chức năng nghiệp vụ của hệ thống.
- *Amazon DynamoDB*: Lưu trữ thông tin người dùng và dữ liệu ứng tuyển.
- *Amazon EventBridge*: Kích hoạt các tác vụ tự động theo lịch. 
- *Amazon SES*: Gửi email nhắc nhở follow-up và thông báo đến người dùng.  
- *Amazon CloudWatch*: Giám sát hiệu năng và ghi nhật ký hoạt động của hệ thống.  
- *AWS CloudTrail*: Ghi nhận lịch sử thao tác và hỗ trợ kiểm tra bảo mật.    

*Thiết kế thành phần*  
- *Thiết bị biên*: Sử dụng Amazon Route 53, AWS WAF và Amazon CloudFront để định tuyến người dùng, bảo vệ hệ thống trước các truy cập độc hại và tăng tốc độ phân phối nội dung.
- *Giao diện người dùng*: Giao diện được phát triển bằng React và triển khai trên Amazon S3, cho phép người dùng quản lý hồ sơ ứng tuyển, CV và theo dõi trạng thái công việc thông qua trình duyệt web.  
- *Quản lý người dùng*: Amazon Cognito đảm nhiệm việc đăng ký, đăng nhập và xác thực người dùng bằng JWT Token, giúp kiểm soát quyền truy cập vào hệ thống.
- *Xử lý dữ liệu*: Amazon API Gateway tiếp nhận các yêu cầu từ người dùng và chuyển đến AWS Lambda để xử lý các chức năng như quản lý công việc, hồ sơ và tạo Presigned URL.
- *Lưu trữ dữ liệu*: Thông tin ứng tuyển được lưu trên Amazon DynamoDB, trong khi các tệp CV và tài liệu được lưu trữ an toàn trên Amazon S3.  
- *Giám sát và vận hành*: Amazon EventBridge, Amazon SES, Amazon CloudWatch và AWS CloudTrail hỗ trợ tự động gửi email nhắc nhở, giám sát hệ thống và ghi nhận nhật ký hoạt động.

### 4. Triển khai kỹ thuật  
*Các giai đoạn triển khai*  
* Triển khai hạ tầng
Hệ thống được triển khai trên nền tảng AWS Serverless, sử dụng Amazon Route 53 để quản lý tên miền, AWS WAF để bảo vệ ứng dụng, Amazon CloudFront để phân phối nội dung và Amazon S3 để lưu trữ giao diện web. Kiến trúc này giúp hệ thống có khả năng mở rộng linh hoạt và giảm chi phí vận hành.

* Triển khai ứng dụng
Giao diện người dùng được phát triển bằng React và kết nối với Amazon API Gateway để gửi yêu cầu đến các AWS Lambda. Người dùng đăng nhập thông qua Amazon Cognito, sau đó sử dụng JWT Token để truy cập các chức năng của hệ thống.

* Triển khai dữ liệu
Dữ liệu về người dùng và quá trình ứng tuyển được lưu trữ trên Amazon DynamoDB, trong khi các tệp CV được lưu trên Amazon S3 dưới dạng Private Bucket. Việc tách dữ liệu và tệp tin giúp hệ thống dễ quản lý, đảm bảo an toàn và nâng cao hiệu suất truy cập.

* Giám sát và bảo mật
Hệ thống sử dụng Amazon EventBridge để kích hoạt các tác vụ tự động, Amazon SES để gửi email nhắc nhở, Amazon CloudWatch để theo dõi hiệu năng và AWS CloudTrail để ghi nhận lịch sử hoạt động. Đồng thời, AWS WAF giúp bảo vệ ứng dụng trước các truy cập trái phép và các cuộc tấn công phổ biến.

* Tóm tắt công nghệ sử dụng

| Thành phần | Công nghệ |
|------------|-----------|
| Frontend | React, HTML, CSS, JavaScript |
| Hosting | Amazon S3, Amazon CloudFront |
| Domain | Amazon Route 53 |
| Authentication | Amazon Cognito |
| API | Amazon API Gateway |
| Business Logic | AWS Lambda |
| Database | Amazon DynamoDB |
| File Storage | Amazon S3 |
| Notification | Amazon SES |
| Scheduler | Amazon EventBridge |
| Monitoring | Amazon CloudWatch, AWS CloudTrail |
| Security | AWS WAF |

---

* Quy trình triển khai
1. Phát triển giao diện người dùng bằng React.
2. Triển khai giao diện lên Amazon S3 và phân phối qua CloudFront.
3. Cấu hình tên miền bằng Amazon Route 53.
4. Thiết lập xác thực người dùng với Amazon Cognito.
5. Xây dựng REST API bằng Amazon API Gateway.
6. Phát triển các hàm AWS Lambda xử lý nghiệp vụ.
7. Lưu trữ dữ liệu trên Amazon DynamoDB và CV trên Amazon S3.
8. Thiết lập EventBridge và Amazon SES để gửi email nhắc nhở.
9. Cấu hình CloudWatch, CloudTrail và AWS WAF để giám sát và bảo mật hệ thống.

*Yêu cầu kỹ thuật*  
| Hạng mục | Yêu cầu kỹ thuật |
|:----------|:-----------------|
| **Nền tảng** | Amazon Web Services (AWS) |
| **Frontend** | React, HTML, CSS, JavaScript |
| **Backend** | AWS Lambda, Amazon API Gateway |
| **Cơ sở dữ liệu** | Amazon DynamoDB |
| **Lưu trữ** | Amazon S3 (Private Bucket) |
| **Xác thực người dùng** | Amazon Cognito (JWT) |
| **Mạng & Phân phối nội dung** | Amazon Route 53, Amazon CloudFront |
| **Bảo mật** | AWS WAF |
| **Tự động hóa** | Amazon EventBridge, Amazon SES |
| **Giám sát hệ thống** | Amazon CloudWatch, AWS CloudTrail |
| **Công cụ phát triển** | Visual Studio Code, Git, GitHub, Node.js |

---  

### 5. Lộ trình & Mốc triển khai

- *Trước thực tập (Tháng 0)*: Dành 1 tháng để khảo sát yêu cầu, lập kế hoạch và thiết kế kiến trúc giải pháp.
- *Thực tập (Tháng 1–3)*:
    - **Tháng 1**: Tìm hiểu các dịch vụ AWS, xây dựng giao diện người dùng bằng React.
    - **Tháng 2**: Triển khai Backend, tích hợp các dịch vụ AWS và xây dựng cơ sở dữ liệu.
    - **Tháng 3**: Triển khai hệ thống, kiểm thử, tối ưu hiệu năng và đưa vào sử dụng.
- *Sau triển khai*: Tiếp tục theo dõi, tối ưu và phát triển thêm các tính năng mới trong vòng 1 năm. 

### 6. Ước tính ngân sách  
Có thể xem chi phí trên [AWS Pricing Calculator](https://calculator.aws/#/estimate?id=621f38b12a1ef026842ba2ddfe46ff936ed4ab01)  
Hoặc tải [tệp ước tính ngân sách](../attachments/budget_estimation.pdf).  

*Chi phí hạ tầng*  
| Dịch vụ AWS | Mục đích sử dụng | Chi phí (USD/tháng) |
|:------------|:-----------------|--------------------:|
| Amazon Route 53 | Quản lý tên miền | $0.50 |
| Tên miền (.com) | Truy cập website | $1.20 |
| Amazon CloudFront | Phân phối nội dung | $1.50 |
| Amazon S3 | Lưu trữ website và CV | $1.00 |
| Amazon Cognito | Xác thực người dùng | $0.00 |
| Amazon API Gateway | Quản lý API | $1.00 |
| AWS Lambda | Xử lý nghiệp vụ | $0.50 |
| Amazon DynamoDB | Lưu trữ dữ liệu | $1.50 |
| Amazon EventBridge | Tự động hóa tác vụ | $0.00 |
| Amazon SES | Gửi email nhắc nhở | $0.50 |
| AWS WAF | Bảo vệ ứng dụng | $8.00 |
| Amazon CloudWatch | Giám sát hệ thống | $2.00 |
| AWS CloudTrail | Ghi nhật ký hoạt động | $0.00 |

*Tổng*: 17.70 USD/tháng, 212,40 USD/12 tháng  
- *Lưu ý*: Đây là chi phí ước tính dành cho môi trường demo hoặc dự án học tập. Nếu lượng người dùng và lưu lượng truy cập tăng, các dịch vụ như CloudFront, Lambda, API Gateway, DynamoDB và S3 sẽ tăng chi phí theo mức sử dụng thực tế. AWS áp dụng mô hình Pay-as-you-go, tức chỉ thanh toán cho tài nguyên đã sử dụng. 

### 7. Đánh giá rủi ro  
*Rủi ro về bảo mật*  
Hệ thống có nguy cơ bị truy cập trái phép hoặc khai thác lỗ hổng bảo mật nếu cơ chế xác thực không được cấu hình đúng. Điều này có thể dẫn đến việc dữ liệu người dùng hoặc các tệp CV bị truy cập trái phép. Để giảm thiểu rủi ro, hệ thống sử dụng Amazon Cognito để xác thực người dùng, JWT Token để kiểm soát quyền truy cập và AWS WAF để ngăn chặn các cuộc tấn công phổ biến từ bên ngoài.

*Rủi ro về hiệu năng*  
Khi số lượng người dùng hoặc số lượng yêu cầu tăng đột biến, hệ thống có thể gặp tình trạng tăng thời gian phản hồi hoặc phát sinh lỗi xử lý. Điều này ảnh hưởng trực tiếp đến trải nghiệm của người dùng. Kiến trúc AWS Serverless cho phép các dịch vụ như AWS Lambda và Amazon DynamoDB tự động mở rộng tài nguyên để đáp ứng nhu cầu sử dụng.

*Rủi ro về lưu trữ dữ liệu*  
Dữ liệu ứng tuyển và các tệp CV là những thông tin quan trọng cần được bảo vệ. Nếu xảy ra lỗi cấu hình hoặc phân quyền không hợp lý, dữ liệu có thể bị mất hoặc truy cập trái phép. Hệ thống sử dụng Amazon S3 Private Bucket, Amazon DynamoDB và cơ chế phân quyền của AWS nhằm đảm bảo tính an toàn và toàn vẹn của dữ liệu.

*Rủi ro về chi phí*  
Do AWS áp dụng mô hình thanh toán theo mức sử dụng, chi phí có thể tăng khi số lượng người dùng và lưu lượng truy cập lớn hơn dự kiến. Nếu không theo dõi thường xuyên, ngân sách vận hành có thể vượt kế hoạch. Để kiểm soát vấn đề này, hệ thống sẽ sử dụng AWS Billing và CloudWatch để giám sát mức sử dụng tài nguyên và tối ưu chi phí.

*Rủi ro trong quá trình vận hành*  
Trong quá trình triển khai hoặc cập nhật hệ thống có thể phát sinh lỗi cấu hình, lỗi mã nguồn hoặc gián đoạn dịch vụ. Những sự cố này có thể làm ảnh hưởng đến hoạt động của người dùng trong một khoảng thời gian nhất định. Để giảm thiểu rủi ro, hệ thống được kiểm thử trước khi triển khai, đồng thời sử dụng CloudWatch và CloudTrail để giám sát, ghi nhận nhật ký và hỗ trợ khắc phục sự cố nhanh chóng.

### 8. Kết quả kỳ vọng  
*Hoàn thành hệ thống*: Xây dựng thành công Job Tracker Platform trên nền tảng AWS Serverless, hỗ trợ quản lý quá trình ứng tuyển việc làm một cách tập trung. Hệ thống đáp ứng đầy đủ các chức năng như quản lý công việc, lưu trữ CV, theo dõi trạng thái ứng tuyển và gửi email nhắc nhở.

*Nâng cao trải nghiệm người dùng*: Người dùng có thể dễ dàng theo dõi tiến độ ứng tuyển và quản lý hồ sơ trên một giao diện trực quan. Chức năng nhắc lịch tự động giúp hạn chế bỏ lỡ các mốc thời gian quan trọng.

*Đảm bảo hiệu năng và bảo mật*: Hệ thống có khả năng mở rộng linh hoạt nhờ kiến trúc AWS Serverless và đảm bảo an toàn dữ liệu thông qua các dịch vụ như Amazon Cognito, AWS WAF và CloudWatch. Điều này giúp hệ thống vận hành ổn định và bảo mật.

*Khả năng phát triển*: Kiến trúc được thiết kế theo hướng dễ mở rộng, cho phép bổ sung các tính năng mới như thống kê dữ liệu, phân tích ứng tuyển và tích hợp AI trong tương lai. Hệ thống có thể đáp ứng nhu cầu phát triển lâu dài mà không cần thay đổi kiến trúc tổng thể.