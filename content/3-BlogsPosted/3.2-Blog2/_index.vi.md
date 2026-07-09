---
title: "Blog 2"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# [SECURITY/Architecture] KIỂM SOÁT TRUY CẬP AN TOÀN CHO ỨNG DỤNG RAG ĐA NGƯỜI DÙNG VỚI AMAZON BEDROCK VÀ VERIFIED PERMISSIONS

### Giới thiệu

Trong quá trình nghiên cứu về các kiến trúc triển khai ứng dụng Generative AI trên Amazon Web Services (AWS), mình có tham khảo bài viết về mô hình **Secure Access Control for Multi-user RAG Applications** trên AWS Architecture Blog. Bài viết tập trung vào cách xây dựng cơ chế phân quyền tập trung cho các ứng dụng Retrieval-Augmented Generation (RAG), giúp kiểm soát quyền truy cập dữ liệu theo từng người dùng mà vẫn duy trì một cơ sở tri thức (Knowledge Base) dùng chung.

Bài viết này tổng hợp và phân tích kiến trúc được AWS đề xuất, đồng thời làm rõ vai trò của từng thành phần trong việc xây dựng hệ thống RAG đa người dùng theo hướng **Security by Design**, đảm bảo tính bảo mật, khả năng mở rộng và dễ dàng quản trị.


### Bối cảnh

Các ứng dụng RAG ngày càng được triển khai rộng rãi trong doanh nghiệp nhằm khai thác tài liệu nội bộ bằng mô hình ngôn ngữ lớn (Large Language Models - LLM).

Tuy nhiên, bài toán quan trọng nhất không nằm ở khả năng tìm kiếm dữ liệu mà là **kiểm soát quyền truy cập**.

Ví dụ:

- Nhân viên phòng Nhân sự chỉ được phép truy cập tài liệu của phòng Nhân sự.
- Phòng Kinh doanh chỉ được phép truy cập tài liệu kinh doanh.
- Ban lãnh đạo có thể được phép truy cập nhiều nhóm dữ liệu hơn.

Một cách triển khai phổ biến là tạo nhiều Knowledge Base riêng biệt cho từng phòng ban. Tuy nhiên, phương pháp này làm tăng:

- Chi phí lưu trữ.
- Chi phí vận hành.
- Độ phức tạp khi quản lý.
- Khả năng mở rộng hệ thống.

AWS đề xuất một hướng tiếp cận khác: sử dụng **một Knowledge Base duy nhất** kết hợp với cơ chế phân quyền động thông qua **Amazon Verified Permissions**.


### 1. Kiểm soát truy cập ở tầng API

#### Vấn đề

Nếu mọi yêu cầu đều được chuyển trực tiếp đến Backend hoặc Knowledge Base, hệ thống sẽ khó kiểm soát các truy cập trái phép ngay từ đầu.

Điều này làm tăng nguy cơ:

- Truy cập API không hợp lệ.
- Tấn công dò quét API.
- Gia tăng tải xử lý cho hệ thống phía sau.

#### Phân tích

AWS triển khai lớp kiểm soát đầu tiên tại **Amazon API Gateway**.

Mỗi request sẽ được:

- Xác thực bằng Amazon Cognito.
- Chuyển đến Lambda Authorizer.
- Lambda Authorizer gửi yêu cầu kiểm tra tới Amazon Verified Permissions.

Nếu người dùng không được cấp quyền, request sẽ bị từ chối trước khi đi vào hệ thống.

#### Lợi ích

- Chặn truy cập trái phép ngay tại Gateway.
- Giảm tải cho Backend.
- Áp dụng nguyên tắc Least Privilege.
- Giảm bề mặt tấn công.

### 2. Kiểm soát quyền truy cập tài liệu

#### Vấn đề

Ngay cả khi người dùng đã được phép sử dụng API, không phải mọi tài liệu đều được phép truy cập.

Nếu LLM truy xuất toàn bộ Knowledge Base, nguy cơ rò rỉ dữ liệu giữa các phòng ban sẽ rất lớn.

#### Phân tích

AWS bổ sung lớp bảo vệ thứ hai thông qua **Middleware Lambda**.

Middleware tiếp tục gửi yêu cầu tới Amazon Verified Permissions để xác định:

- Người dùng thuộc nhóm nào.
- Được phép truy cập loại tài liệu nào.
- Metadata Filter nào cần được tạo.

Metadata Filter này sẽ được truyền trực tiếp tới API **RetrieveAndGenerate** của Amazon Bedrock.

Nhờ đó, mô hình chỉ truy xuất những tài liệu mà người dùng được phép xem.


### Hình 1. Kiến trúc kiểm soát truy cập cho ứng dụng RAG đa người dùng

![Kiến trúc RAG với Amazon Verified Permissions](/images/h1.2.jpg)

**Hình 1** minh họa kiến trúc kiểm soát truy cập nhiều lớp cho ứng dụng RAG trên AWS.

Luồng hoạt động gồm các bước chính:

1. Người dùng truy cập ứng dụng thông qua Amazon CloudFront.
2. CloudFront phân phối giao diện Web được lưu trữ trên Amazon S3.
3. Amazon Cognito thực hiện xác thực người dùng và cấp JWT Token.
4. Request được chuyển tới AWS WAF để kiểm tra các mối đe dọa ở tầng Web.
5. AWS API Gateway tiếp nhận yêu cầu và chuyển tới Lambda Authorizer.
6. Lambda Authorizer gửi yêu cầu xác thực quyền đến Amazon Verified Permissions.
7. Amazon Verified Permissions đánh giá chính sách Cedar và trả về quyết định cho phép hoặc từ chối.
8. Nếu được cấp quyền, API Gateway chuyển request đến Middleware Lambda.
9. Middleware Lambda tiếp tục truy vấn Verified Permissions để tạo Metadata Filter phù hợp với người dùng.
10. Middleware gọi Amazon Bedrock Knowledge Bases để truy xuất dữ liệu phù hợp.
11. Amazon Bedrock chỉ tìm kiếm trên tập tài liệu được lọc từ Amazon S3 Vectors trước khi sinh câu trả lời.

Kiến trúc này tạo thành hai lớp bảo vệ độc lập, giúp giảm nguy cơ truy cập trái phép và hạn chế rò rỉ dữ liệu giữa các nhóm người dùng.


### 3. Quản lý chính sách tập trung với Amazon Verified Permissions

#### Vấn đề

Nếu logic phân quyền được viết trực tiếp trong mã nguồn, mỗi lần thay đổi quyền truy cập đều yêu cầu sửa mã và triển khai lại ứng dụng.

Điều này làm tăng:

- Chi phí bảo trì.
- Khả năng phát sinh lỗi.
- Độ phức tạp trong quản lý.

#### Phân tích

Amazon Verified Permissions sử dụng ngôn ngữ **Cedar** để mô tả chính sách phân quyền.

Mọi chính sách được quản lý tập trung và tách biệt hoàn toàn khỏi mã nguồn ứng dụng.

Khi có thay đổi về:

- Người dùng.
- Vai trò.
- Phòng ban.
- Chính sách truy cập.

Quản trị viên chỉ cần cập nhật chính sách trên dịch vụ mà không cần triển khai lại hệ thống.

#### Lợi ích

- Quản lý tập trung.
- Dễ mở rộng.
- Dễ kiểm toán.
- Tuân thủ nguyên tắc Deny-by-default.

### 4. Kiến trúc Defense-in-Depth cho ứng dụng GenAI

#### Phân tích

Kiến trúc được AWS đề xuất áp dụng mô hình **Defense in Depth**, trong đó mỗi tầng đảm nhận một nhiệm vụ bảo vệ riêng.

Các lớp bảo vệ gồm:

- Amazon Cognito xác thực người dùng.
- AWS WAF bảo vệ tầng Web.
- API Gateway kiểm soát điểm vào.
- Lambda Authorizer xác thực quyền.
- Amazon Verified Permissions quyết định quyền truy cập.
- Middleware Lambda tạo Metadata Filter.
- Amazon Bedrock chỉ truy cập dữ liệu được cấp phép.

Việc kiểm tra quyền ở nhiều tầng giúp giảm khả năng bỏ sót lỗi cấu hình và hạn chế tối đa nguy cơ rò rỉ dữ liệu.

### Bảng tổng hợp kiến trúc

| Thành phần | Dịch vụ AWS | Vai trò |
|------------|-------------|---------|
| Authentication | Amazon Cognito | Xác thực người dùng |
| Edge Protection | AWS WAF | Bảo vệ ứng dụng Web |
| API Layer | Amazon API Gateway | Tiếp nhận và quản lý API |
| Authorization | Amazon Verified Permissions | Kiểm tra quyền truy cập |
| Policy Engine | Cedar | Quản lý chính sách phân quyền |
| AI Platform | Amazon Bedrock Knowledge Bases | Truy xuất dữ liệu cho RAG |
| Data Storage | Amazon S3 Vectors | Lưu trữ dữ liệu vector |

### Kết luận

Kiến trúc được AWS đề xuất cho thấy việc xây dựng ứng dụng Generative AI an toàn không chỉ phụ thuộc vào mô hình ngôn ngữ lớn mà còn phụ thuộc vào cách kiểm soát quyền truy cập dữ liệu.

Việc kết hợp Amazon Cognito, AWS WAF, API Gateway, Amazon Verified Permissions và Amazon Bedrock tạo nên một kiến trúc nhiều lớp giúp giảm nguy cơ truy cập trái phép, hạn chế rò rỉ dữ liệu giữa các phòng ban và tăng khả năng mở rộng của hệ thống.

Đây là một hướng tiếp cận phù hợp cho các doanh nghiệp triển khai ứng dụng RAG đa người dùng theo tư duy **Security by Design** và **Defense in Depth**.


### Tài liệu tham khảo

https://aws.amazon.com/vi/blogs/architecture/secure-multi-tenant-rag-with-amazon-bedrock-and-verified-permissions/

### Link bài viết

https://www.facebook.com/groups/awsstudygroupfcj/permalink/2202713613826932/?rdid=moGmZYTchQTj5BBA#