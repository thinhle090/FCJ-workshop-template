---
title: "Sự kiện 2 - AWS Vietnam Community Day 2026"
date: 2026-05-23
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---

| Thông tin | Chi tiết |
|---|---|
| Ngày | 23/05/2026 |
| Địa điểm | Tầng 26, Tòa nhà Bitexco Financial Tower, Phường Sài Gòn, TP.Hồ Chí Minh |
| Vai trò | Người tham dự |

Trang này tổng hợp nội dung các bài chia sẻ tại **AWS Vietnam Community Day 2026**, bao gồm thiết kế context AI, hạ tầng edge, kinh nghiệm hackathon, độ tin cậy của LLM, và hệ thống multi-agent cấp doanh nghiệp.

---

### Trọng tâm chung

> AI không chỉ là công cụ demo. AI cần có bối cảnh, kiến trúc và quy trình triển khai rõ ràng.

- AWS đóng vai trò nền tảng cho hạ tầng, bảo mật, vận hành và mở rộng hệ thống AI
- Các bài chia sẻ đều nhấn mạnh tính thực tế: từ prompt, hackathon, CloudFront đến enterprise-grade multi-agent

---

### 1. Giới thiệu

AWS Vietnam Community Day 2026 quy tụ các chuyên gia chia sẻ kinh nghiệm thực tế trong việc xây dựng hệ thống AI và cloud trên AWS. Các chủ đề trải rộng từ năng suất cá nhân với AI, công cụ trợ lý doanh nghiệp, hạ tầng CDN, phát triển sản phẩm trong hackathon, vấn đề độ tin cậy của LLM, đến hệ thống chấm điểm tín dụng multi-agent cấp enterprise.

---

### 2. Tóm tắt các bài chia sẻ

#### 2.1 Tinh Truong - Build Second Brain

Bài chia sẻ tập trung vào cách làm việc hiệu quả với AI thông qua **quản lý context**. Diễn giả nhấn mạnh rằng các AI model hiện nay đã rất mạnh, nhưng kết quả vẫn thường kém vì người dùng hoặc không cung cấp đủ bối cảnh, hoặc đưa vào những bối cảnh sai trọng tâm.

**Một context tốt cần có:**
- Mục tiêu cần đạt được
- Tình huống hiện tại
- Ràng buộc kỹ thuật
- Bằng chứng hoặc dữ liệu liên quan

**Sai lầm phổ biến:**
- Đưa quá nhiều tài liệu không chọn lọc
- Sao chép nguyên các file dài
- Chỉ nêu lại những điều hiển nhiên mà AI đã biết

**Nguyên tắc cốt lõi:**
> Chất lượng context quan trọng hơn số lượng context.

**Khái niệm Second AI Brain:**
Đây là hệ thống tổ chức tri thức cá nhân, giúp bạn nhớ lại dự án và truy xuất đúng thông tin trước khi đặt câu hỏi cho AI.

**Bài học rút ra:** Người dùng AI giỏi là người biết chuyển một yêu cầu mơ hồ thành một tác vụ có mục tiêu, dữ liệu và đầu ra rõ ràng.

---

#### 2.2 Phạm Nguyễn Hải Anh - Friendly AI Assistant with Amazon Quick Suite

Bài trình bày đề cập đến các khó khăn phổ biến của người dùng doanh nghiệp và quản lý dự án: quản lý quá nhiều tài liệu, cuộc họp, email, dữ liệu và các tác vụ lặp lại. **Amazon Quick Suite** được giới thiệu như một trợ lý AI xây dựng trên Bedrock, tìm kiếm web và dữ liệu nội bộ để tối ưu các quy trình làm việc này.

**Amazon Quick Suite hỗ trợ:**
- Chat và Q&A thông minh
- Nghiên cứu và tìm kiếm thông minh
- Dashboard BI
- Tự động hóa quy trình
- Nhúng API vào quy trình doanh nghiệp
- Nền tảng: Amazon Bedrock + Tìm kiếm web + Dữ liệu nội bộ

**Tình huống minh họa:**
Một trợ lý AI cho quản lý dự án có thể tự động tạo biên bản họp, gửi email cho các bên liên quan và lên lịch cho cuộc họp tiếp theo.

**Giá trị chính:**
- Giảm thời gian dành cho các tác vụ lặp lại và việc thu thập thông tin
- Giúp người dùng tập trung hơn vào quyết định và phối hợp nhóm

**Bài học rút ra:** AI tạo ra giá trị lớn nhất khi được đặt đúng vào quy trình làm việc, hiểu dữ liệu nội bộ và hỗ trợ hành động tiếp theo.

---

#### 2.3 Nguyễn Tuấn Thịnh - Từ Edge đến Origin: CloudFront là nền tảng của bạn

Bài chia sẻ tập trung vào **Amazon CloudFront** như một lớp nền tảng hoàn chỉnh từ edge đến origin, bao quát chi phí, bảo mật, hiệu năng và độ tin cậy ở quy mô lớn.

**Mô hình chi phí:**
- Gói giá cố định bao gồm CDN, WAF, DDoS, DNS và logging
- Mức giá dễ dự đoán, phù hợp với chủ website nhỏ, người dùng doanh nghiệp và các doanh nghiệp đang mở rộng
- Xử lý được lưu lượng tăng đột biến mà không gây bùng nổ chi phí bất ngờ

**Các tính năng bảo mật:**

| Tính năng | Mô tả |
|---|---|
| DDoS Protection | Bảo vệ trước các cuộc tấn công lưu lượng lớn |
| WAF | Web Application Firewall |
| DNS | Tích hợp với Route 53 |
| TLS / mTLS | TLS miễn phí cùng mutual TLS cho kết nối được mã hóa |
| Signed URL | Phân phối nội dung có kiểm tra xác thực |
| Origin Cloaking | Ẩn máy chủ gốc khỏi truy cập công khai |

**Tối ưu hiệu năng:**
- Bộ nhớ đệm nhiều lớp tại edge để giảm tải origin và tối ưu băng thông
- Hỗ trợ HTTP/3
- Nén dữ liệu
- Kết nối liên tục để giảm tải origin
- Edge functions cho các logic yêu cầu độ trễ thấp

**Các yếu tố đảm bảo độ tin cậy:**
- Phục vụ nội dung cũ khi origin gặp sự cố
- Chuyển đổi dự phòng origin
- Định tuyến thông minh

**Bài học rút ra:** CloudFront không chỉ là một CDN. Đây là lớp nền tảng cho tối ưu chi phí, bảo mật, hiệu năng và khả năng chịu lỗi.

---

#### 2.4 Team VIB - 36 Giờ với LotusHacks: Xây dựng UTMorpho từ Ý tưởng đến Hiện thực

Team VIB chia sẻ câu chuyện thực tế khi tham gia **hackathon LotusHacks** trong 36 giờ, từ chỗ chưa có ý tưởng đến lúc hoàn thành demo hoạt động của **UTMorpho**.

**UTMorpho làm gì:**
Người dùng có thể chụp ảnh, vẽ hoặc tải lên bản phác thảo UI, sau đó AI sẽ tạo giao diện web từ đầu vào đó.

**Kiến trúc sử dụng:**
```text
CloudFront -> API Gateway -> Lambda -> Bedrock -> S3 / DynamoDB
```

**Pipeline AI agent:**
1. **Vision Analyst** - Phân tích bản phác thảo đầu vào
2. **UI Designer** - Chuyển thành bản đặc tả thiết kế
3. **Coder** - Sinh ra mã nguồn thực tế

**Những thách thức chính:**
- Giới hạn token từ context window của LLM
- AI sinh quá nhiều nội dung dẫn đến đầu ra bị nhiễu
- Áp lực thời gian thuyết trình
- Phạm vi dự án bị phình ra do có quá nhiều ý tưởng

**Bài học rút ra:** Những khó chịu thực tế mới tạo ra ý tưởng thực tế. Hackathon đòi hỏi phối hợp nhóm tốt, kiểm soát phạm vi chặt và tập trung vào một trải nghiệm cốt lõi thực sự hữu ích.

---

#### 2.5 Đào Đức - Tính không tất định của các thiết lập LLM tất định

Bài trình bày này trả lời một câu hỏi kỹ thuật quan trọng: **Vì sao LLM vẫn có thể cho ra kết quả khác nhau ngay cả khi đặt `temperature=0`?** Đây là vấn đề rất đáng lưu ý với các hệ thống có mức độ rủi ro cao như pháp lý, tài chính hoặc truy xuất thông tin y tế.

**Cách LLM sinh token:**
- Logit computation -> Softmax -> Sampling
- `temperature` chỉ điều chỉnh phân phối xác suất, chứ không loại bỏ hoàn toàn nguồn gốc của non-determinism

**Các thử nghiệm đã thực hiện:**
- **5 model được kiểm thử:** GPT-3.5, GPT-4o, Llama-3 70B, Llama-3 8B, Mixtral 8x7B
- **8 tác vụ x 10 lần chạy** cho mỗi model; kết quả cho thấy độ chính xác dao động đáng kể giữa các lần chạy giống hệt nhau

**Nguyên nhân kỹ thuật:**

| Nguyên nhân | Giải thích |
|---|---|
| Phép tính dấu phẩy động | Các phép tính trên GPU không hoàn toàn tất định |
| Thứ tự thực thi song song | Thứ tự lập lịch của luồng GPU có thể thay đổi |
| Gom lô suy luận | Việc gom lô phía nhà cung cấp làm thay đổi thứ tự xử lý |

**Chiến lược giảm thiểu:**
- Chạy nhiều lần và dùng phương pháp **bỏ phiếu đa số**
- Dùng **đầu ra có cấu trúc** như JSON, regex hoặc ràng buộc theo ngữ pháp
- Áp dụng **kiểm thử hồi quy** để kiểm tra độ ổn định của đầu ra
- **Tự triển khai** model khi cần kiểm soát hoàn toàn quá trình suy luận
- Thiết kế hệ thống **chịu được dao động** ngay từ đầu

**Điểm cân bằng:** `temperature ~= 0.1` cho sự cân bằng tốt hơn giữa độ ổn định và chất lượng output so với việc cố định tuyệt đối ở `temperature=0`.

**Bài học rút ra:** `temperature=0` không phải là đảm bảo cho độ tin cậy. Hệ thống cần được thiết kế để xử lý sự dao động ngay từ đầu.

---

#### 2.6 Vy Lam - Hệ thống multi-agent cấp doanh nghiệp: Chấm điểm tín dụng cho startup

Bài chia sẻ trình bày một **hệ thống chấm điểm tín dụng multi-agent** cho startup, một lĩnh vực mà mô hình đánh giá tín dụng truyền thống thường thất bại vì dữ liệu của startup khác biệt căn bản so với doanh nghiệp đã vận hành lâu năm.

**Vì sao chấm điểm tín dụng truyền thống không phù hợp với startup:**
- Cần lịch sử tài chính dài, tài sản thế chấp và mô hình doanh thu ổn định
- Startup thường chỉ có traction, chất lượng đội ngũ, IP và dữ liệu phi cấu trúc

**Các chiều dữ liệu của startup:**

| Chiều | Ví dụ |
|---|---|
| Tài chính | Doanh thu, tốc độ đốt vốn, thời gian sống |
| Thị trường | Quy mô thị trường, bối cảnh cạnh tranh |
| Đội ngũ | Kinh nghiệm, nền tảng, sự đa dạng |
| Sức kéo | Tăng trưởng người dùng, tỷ lệ giữ chân, quan hệ đối tác |

**Thiết kế hệ thống - Hội đồng tín dụng ảo:**

| Agent | Trách nhiệm |
|---|---|
| Quản lý | Điều phối toàn bộ quá trình đánh giá |
| Phân tích tài chính | Đánh giá các chỉ số tài chính |
| Phân tích thị trường | Đánh giá cơ hội thị trường |
| Đánh giá đội ngũ | Rà soát đội ngũ sáng lập |
| Đánh giá rủi ro | Xác định các yếu tố rủi ro |
| Kiểm tra tuân thủ | Đảm bảo tuân thủ quy định |

**Yêu cầu đầu ra:**
- Điểm tín dụng
- Mức xếp hạng rủi ro
- Mức độ tin cậy
- Nhật ký kiểm toán với khả năng giải thích quyết định

**Các cân nhắc ở cấp doanh nghiệp - 6 trụ cột:**

| Trụ cột | Phạm vi |
|---|---|
| Bảo mật | Xác thực, phân quyền, mã hóa |
| Quản trị dữ liệu | Truy vết dữ liệu, kiểm soát truy cập, lưu trữ |
| Mạng | Cô lập VPC, private endpoints |
| Vận hành | Giám sát, cảnh báo, xử lý sự cố |
| Yếu tố con người | Khả năng giải thích, quy trình phê duyệt |
| Tuân thủ | Phù hợp quy định, sẵn sàng kiểm toán |

**Rào chắn an toàn - Ba tầng (Đầu vào -> Xử lý -> Đầu ra):**

| Tầng | Kiểm soát |
|---|---|
| Đầu vào | Lọc nội dung, phát hiện thông tin cá nhân, chống tấn công injection |
| Xử lý | Kiểm soát lựa chọn model, ràng buộc suy luận |
| Đầu ra | Xác thực phản hồi, kiểm tra tuân thủ |

**Lộ trình triển khai:**
```text
Local App / CrewAI -> AgentCore -> Docker -> ECR -> Bedrock -> API Gateway
                    + VPC, IAM, Secrets, Monitoring, Autoscaling, DR Strategy
```

**ROI kỳ vọng:**
- Thời gian xử lý: từ vài tuần xuống vài giờ
- Giảm số giờ làm việc của chuyên viên phân tích
- Tăng độ chính xác phê duyệt nhờ đánh giá đa chiều

---

![Hình ảnh tại Event 2](/images/b2.jpg)

---

### 4. Kết luận

AWS Vietnam Community Day 2026 cho thấy giá trị của AI không nằm ở riêng model, mà còn nằm ở cách cung cấp bối cảnh, thiết kế kiến trúc, triển khai bảo mật và rào chắn an toàn, cũng như vận hành hệ thống ở quy mô thực tế.

| Đối tượng | Bài học chính |
|---|---|
| Cá nhân | Học cách cung cấp bối cảnh chất lượng và xây dựng kho tri thức cá nhân để làm việc hiệu quả hơn với AI |
| Đội nhóm sản phẩm | Ưu tiên bài toán thực, giới hạn phạm vi và đưa AI vào quy trình cụ thể |
| Hạ tầng | Tận dụng CloudFront và các dịch vụ AWS cho hiệu năng, bảo mật và độ tin cậy |
| Doanh nghiệp | Hệ thống multi-agent cần rào chắn an toàn, nhật ký kiểm toán, tuân thủ quy định và ROI rõ ràng trước khi đưa vào vận hành thực tế |

> **Thông điệp chung:** AI chỉ thực sự tạo ra giá trị khi được kết hợp với tư duy sản phẩm, kiến trúc hệ thống phù hợp và quy trình vận hành đáng tin cậy.
