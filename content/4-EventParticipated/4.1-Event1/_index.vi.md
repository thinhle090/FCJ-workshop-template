---
title: "Sự kiện 1 - Prompt Engineering, Sơ đồ tư duy AI & Phương pháp BMAD"
date: 2026-05-09
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

| Thông tin | Chi tiết |
|---|---|
| Ngày | 09/05/2026 |
| Địa điểm | Tầng 26, Tòa nhà Bitexco Financial Tower, Phường Sài Gòn, TP. Hồ Chí Minh |
| Vai trò | Người tham dự |

Trang này tổng hợp những nội dung nổi bật được chia sẻ trong sự kiện, tập trung vào ba chủ đề chính gồm: **Automated Prompt Engineering**, sơ đồ tư duy về cách tương tác hiệu quả với AI và phương pháp phát triển **BMAD (Build More Architect Dreams)**.

---

### 1. Giới thiệu

Nội dung được tổng hợp từ nhiều tài liệu và phần trình bày tại sự kiện, bao gồm:

- Bài thuyết trình **"Automated Prompt Engineering: Enhancing LLM Output Quality"**
- Sơ đồ tư duy **"Tương tác hiệu quả với AI & Kiến trúc ứng dụng AWS"**
- Phương pháp phát triển AI **BMAD (Build More Architect Dreams)**

---

### 2. Nội dung chi tiết

#### 2.1 Bài thuyết trình: Automated Prompt Engineering

**Diễn giả: Nguyễn Tuấn Thịnh**

Bài trình bày giới thiệu vai trò của Prompt Engineering trong việc nâng cao chất lượng phản hồi của các mô hình ngôn ngữ lớn (LLM). Đồng thời, diễn giả chia sẻ nhiều nguyên tắc và kỹ thuật giúp xây dựng prompt hiệu quả hơn.

**Tầm quan trọng của Prompt Engineering:**

- Prompt quá chung chung thường tạo ra kết quả chưa chính xác.
- Tiêu tốn nhiều token không cần thiết.
- Hướng dẫn thiếu rõ ràng làm giảm hiệu quả xử lý của AI.

**Các thành phần của một prompt hiệu quả:**

| Thành phần | Mô tả |
|---|---|
| Vai trò | Xác định vai trò hoặc nhân vật mà AI sẽ đảm nhận |
| Hướng dẫn | Mô tả rõ nhiệm vụ AI cần thực hiện |
| Bối cảnh | Cung cấp thông tin nền liên quan |
| Dữ liệu đầu vào | Nội dung AI sẽ xử lý |
| Định dạng đầu ra | Quy định cách trình bày kết quả |
| Ví dụ | Minh họa đầu vào và đầu ra mong muốn |
| Ràng buộc | Các giới hạn hoặc quy tắc cần tuân thủ |

**Một số nguyên tắc khi viết Prompt:**

- Trình bày rõ ràng, cụ thể.
- Sử dụng câu lệnh trực tiếp.
- Mô tả chính xác yêu cầu cần thực hiện.
- Cho phép AI phản hồi "Tôi không biết" khi thiếu dữ liệu.
- Chia các yêu cầu phức tạp thành nhiều bước nhỏ.

**Kinh tế Token (Token Economy):**

- Token là đơn vị dữ liệu được mô hình AI sử dụng để xử lý.
- Chi phí được tính dựa trên số lượng token đầu vào và đầu ra.
- Tiếng Việt thường sử dụng nhiều token hơn so với tiếng Anh.

**Các kỹ thuật nâng cao:**

- **Chain-of-Thought (CoT):** Hướng AI suy luận theo từng bước.
- **Tree-of-Thoughts (ToT):** Xây dựng nhiều hướng suy luận khác nhau.
- **Self-Consistency:** So sánh nhiều kết quả và lựa chọn đáp án phù hợp nhất.
- **Retrieval-Augmented Generation (RAG):** Bổ sung dữ liệu từ nguồn bên ngoài.
- **Role Prompting:** Giao vai trò cụ thể để AI phản hồi đúng ngữ cảnh.

**Công cụ Proptimizer:**

Đây là tiện ích mở rộng trên trình duyệt giúp tối ưu hóa prompt tự động, được triển khai trên kiến trúc AWS Serverless gồm:

CloudFront → S3 → Cognito → API Gateway → Lambda → Bedrock → DynamoDB → CloudWatch

---

#### 2.2 Sơ đồ tư duy: Tương tác AI hiệu quả & Kiến trúc ứng dụng AWS

**Tối ưu cách làm việc với AI:**

Sơ đồ tư duy giới thiệu mô hình **KFC (Knowledge - Format - Constraints)** nhằm giúp người dùng xây dựng prompt đầy đủ ngữ cảnh, đúng định dạng và có các ràng buộc rõ ràng.

**Một số phương pháp được khuyến nghị:**

- Chia bài toán thành nhiều phần nhỏ.
- Yêu cầu AI đề xuất nhiều phương án giải quyết.
- So sánh các câu trả lời để lựa chọn phương án tối ưu.
- Cung cấp dữ liệu đầu vào có cấu trúc rõ ràng.

**Các kỹ thuật AI nâng cao:**

- Chain-of-Thought (CoT)
- Tree-of-Thoughts (ToT)
- Self-Consistency
- Retrieval-Augmented Generation (RAG)
- Role Prompting

**Kiến trúc AWS Serverless:**

CloudFront → S3 → Cognito → API Gateway → Lambda → Bedrock → DynamoDB → CloudWatch

---

#### 2.3 Phương pháp BMAD (Build More Architect Dreams)

BMAD là một framework mã nguồn mở hỗ trợ phát triển các dự án AI theo quy trình chuẩn, giúp nhóm phát triển tổ chức công việc khoa học và hiệu quả hơn.

**Những đặc điểm nổi bật:**

- Quy trình gồm các bước: Phân tích → Lập kế hoạch → Thiết kế kiến trúc → Triển khai.
- Cung cấp hơn 12 AI Agent phục vụ từng giai đoạn phát triển.
- Tích hợp phương pháp Agile vào quy trình làm việc.
- Hỗ trợ trợ lý AI thông qua lệnh **bmad-help**.
- Chế độ **Party Mode** cho phép nhiều Agent phối hợp cùng lúc.

**Các mô-đun chính:**

| Mô-đun | Mô tả |
|---|---|
| BMM | Bộ khung chính với 34 quy trình |
| BMB | Xây dựng AI Agent tùy chỉnh |
| TEA | Hỗ trợ kiểm thử và tự động hóa |
| BMGD | Phát triển trò chơi |
| CIS | Hỗ trợ đổi mới và Design Thinking |

**Cài đặt nhanh** *(Yêu cầu Node.js 20+, Python 3.10+ và uv)*

```bash
npx bmad-method install
```

**Cộng đồng hỗ trợ:**

Discord, YouTube và X (Twitter).

---

### 3. Hình ảnh khi tham gia sự kiện

Dưới đây là một số hình ảnh ghi lại quá trình tham gia sự kiện:

![Hình ảnh tại Event 1](/images/b1.jpg)

---

### 4. Kết luận

Sự kiện mang đến nhiều kiến thức thực tiễn về Prompt Engineering, phương pháp tương tác hiệu quả với AI và quy trình phát triển BMAD. Các nội dung được trình bày rõ ràng, có tính ứng dụng cao trong quá trình phát triển các hệ thống AI cũng như các giải pháp dựa trên nền tảng AWS Serverless. Đây là nguồn tài liệu hữu ích giúp nâng cao kỹ năng làm việc với AI và hỗ trợ triển khai các dự án trong thực tế.