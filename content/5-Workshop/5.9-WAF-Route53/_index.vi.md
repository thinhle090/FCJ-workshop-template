---
title : "Bảo vệ với WAF và quản lý DNS bằng Route 53"
date : "2026-07-07"
weight : 9
chapter : false
pre : " <b> 5.9 </b> "
---

#### Nội dung
- [Bước 1: Bật AWS WAF trên Amplify](#bước-1-bật-aws-waf-trên-amplify)
- [Bước 2: Kiểm thử sau khi bật WAF](#bước-2-kiểm-thử-sau-khi-bật-waf)
- [Bước 3: Tạo Hosted Zone trên Route 53](#bước-3-tạo-hosted-zone-trên-route-53)
- [Bước 4: Trỏ nameserver tại nhà đăng ký](#bước-4-trỏ-nameserver-tại-nhà-đăng-ký)
- [Bước 5: Gắn custom domain vào Amplify](#bước-5-gắn-custom-domain-vào-amplify)

#### Bước 1: Bật AWS WAF trên Amplify

**AWS WAF** lọc traffic độc hại ngay tại edge, trước khi request vào ứng dụng.

1. Vào **AWS Amplify** → chọn app `job-tracker-web`
2. Ở phần **Get to production**, chọn **Enable firewall** (hoặc menu **Hosting → Firewall**)
3. Bật **Enable Amplify-recommended Firewall protection** — bao gồm:
   - Chống các lỗ hổng web phổ biến (SQL injection, XSS)
   - Chống dò quét lỗ hổng ứng dụng
   - Chặn IP độc hại theo threat intelligence của Amazon
4. Các mục **Restrict access to amplifyapp.com**, **IP address protection**, **Country protection**: để **tắt** (không cần cho phạm vi demo)
5. Chọn **Add firewall**

Amplify tạo một **WebACL** (`CreatedByAmplify-...`) với 3 managed rules, scope **CloudFront (Global)**.

![WAF WebACL](/images/5-Workshop/5.9-WAF-Route53/waf-webacl.png)

{{% notice warning %}}
**Chi phí:** Amplify Firewall tính **$15/tháng cho mỗi app**, cộng phí AWS WAF (~$8/tháng + $1.40 cho mỗi 1 triệu request). Đây là khoản đáng kể — nhớ **tắt WAF sau khi hoàn thành workshop** nếu không tiếp tục sử dụng.
{{% /notice %}}

#### Bước 2: Kiểm thử sau khi bật WAF

Managed rules đôi khi chặn nhầm request hợp lệ. Sau khi WAF chuyển trạng thái **Active**, mở URL Amplify và kiểm tra lại toàn bộ:

- [ ] Đăng nhập
- [ ] Tạo job mới (POST có body JSON — dễ bị rule SQLi/XSS soi nhất)
- [ ] Upload CV, tải CV
- [ ] Nhấn F5 giữa trang

Nếu thao tác nào trả **403 Forbidden**: vào **WAF & Shield** → chọn WebACL → **Manage rules** → chuyển rule đang chặn sang chế độ **Count** (theo dõi thay vì chặn). Dùng nút **View WAF logs** để xem chính xác request nào bị block.

#### Bước 3: Tạo Hosted Zone trên Route 53

{{% notice info %}}
Mô hình sử dụng: **mua domain ở nhà đăng ký bên ngoài** (ví dụ Vietnix), còn **Route 53 chỉ làm DNS hosting**. Hai vai trò tách biệt: registrar giữ quyền sở hữu, Route 53 quản lý bản ghi DNS.
{{% /notice %}}

1. Vào **Route 53** → **Hosted zones** → **Create hosted zone**
2. **Domain name**: nhập domain đã mua (ví dụ `jobtracker-fcj.io.vn`)
3. **Type**: **Public hosted zone**
4. Chọn **Create hosted zone**
5. Hosted zone tạo xong có 2 record mặc định: **NS** và **SOA**. Mở record **NS**, copy đủ **4 nameserver**:

```
ns-1817.awsdns-35.co.uk
ns-968.awsdns-57.net
ns-11.awsdns-01.com
ns-1391.awsdns-45.org
```

![Route 53 hosted zone](/images/5-Workshop/5.9-WAF-Route53/route53-hostedzone.png)

**Chi phí:** hosted zone tính **$0.50/tháng**, phí truy vấn DNS không đáng kể.

#### Bước 4: Trỏ nameserver tại nhà đăng ký

1. Đăng nhập trang quản trị của nhà đăng ký (name.com)
2. Vào **Manage** → chọn domain → tab **Nameservers**
3. Chọn **Custom**
4. Xoá nameserver mặc định, dán **4 nameserver của AWS** (không có dấu chấm ở cuối)
5. Chọn **Save**

![name.com nameservers](/images/5-Workshop/5.9-WAF-Route53/name-ns.png)

{{% notice warning %}}
Sau khi đổi nameserver, DNS cần **propagate** từ 15 phút đến vài giờ (đôi khi tới 24–48 giờ). Kiểm tra tiến độ tại [dnschecker.org](https://dnschecker.org) — chọn type **NS**, khi kết quả trả về `awsdns...` là đã hoàn tất.
{{% /notice %}}

#### Bước 5: Gắn custom domain vào Amplify

1. Vào **Amplify** → app → **Hosting** → **Custom domains** → **Add domain**
2. Nhập domain (ví dụ `jobtrackerfcj.dev`) → **Configure domain**
3. Amplify tự cấp **SSL certificate miễn phí** và hiển thị các bản ghi DNS cần thêm
4. Nếu hosted zone cùng account, Amplify tự thêm record vào Route 53; nếu khác account, copy record thủ công vào hosted zone
5. Có thể map cả root domain và subdomain `www`
6. Chờ trạng thái chuyển từ **Pending verification** sang **Available**

Cuối cùng, thêm domain mới vào **CORS** của API Gateway và S3 bucket CV (như Bước 7 mục 5.8).

**Kiểm thử:** mở `https://<domain-của-bạn>` → thấy ổ khoá HTTPS → đăng nhập → thao tác đầy đủ.

{{% notice tip %}}
Amplify **không tính phí** cho custom domain và SSL certificate. Chỉ Route 53 hosted zone tốn $0.50/tháng nếu bạn quản lý DNS qua AWS.
{{% /notice %}}
