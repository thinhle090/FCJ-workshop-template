---
title : "Protect with WAF and Manage DNS with Route 53"
date : "2026-07-07"
weight : 9
chapter : false
pre : " <b> 5.9 </b> "
---

#### Content
- [Step 1: Enable AWS WAF on Amplify](#step-1-enable-aws-waf-on-amplify)
- [Step 2: Test after enabling WAF](#step-2-test-after-enabling-waf)
- [Step 3: Create a Route 53 Hosted Zone](#step-3-create-a-route-53-hosted-zone)
- [Step 4: Point nameservers at the registrar](#step-4-point-nameservers-at-the-registrar)
- [Step 5: Attach the custom domain to Amplify](#step-5-attach-the-custom-domain-to-amplify)

#### Step 1: Enable AWS WAF on Amplify

**AWS WAF** filters malicious traffic at the edge, before requests reach the application.

1. Go to **AWS Amplify** → select the `job-tracker-web` app
2. Under **Get to production**, choose **Enable firewall** (or **Hosting → Firewall**)
3. Turn on **Enable Amplify-recommended Firewall protection**, which covers:
   - Common web vulnerabilities (SQL injection, XSS)
   - Actors probing for application vulnerabilities
   - IP addresses flagged by Amazon threat intelligence
4. Leave **Restrict access to amplifyapp.com**, **IP address protection** and **Country protection** **off** (not needed for this scope)
5. Choose **Add firewall**

Amplify creates a **WebACL** (`CreatedByAmplify-...`) with 3 managed rules, scoped to **CloudFront (Global)**.

![WAF WebACL](/images/5-Workshop/5.9-WAF-Route53/waf-webacl.png)

{{% notice warning %}}
**Cost:** Amplify Firewall charges **$15/month per app**, plus AWS WAF fees (~$8/month + $1.40 per million requests). This is significant — remember to **disable WAF after the workshop** if you are not continuing to use it.
{{% /notice %}}

#### Step 2: Test after enabling WAF

Managed rules occasionally block legitimate requests. Once WAF reaches the **Active** state, open the Amplify URL and re-test everything:

- [ ] Sign in
- [ ] Create a new job (a POST with a JSON body — the most likely target of SQLi/XSS rules)
- [ ] Upload and download a CV
- [ ] Press F5 mid-page

If any action returns **403 Forbidden**: go to **WAF & Shield** → select the WebACL → **Manage rules** → switch the offending rule to **Count** mode (monitor instead of block). Use **View WAF logs** to see exactly which request was blocked.

#### Step 3: Create a Route 53 Hosted Zone

{{% notice info %}}
The model used here: **buy the domain from an external registrar** (e.g. Vietnix) while **Route 53 acts purely as DNS hosting**. The roles are separate: the registrar owns the name, Route 53 manages the DNS records.
{{% /notice %}}

1. Go to **Route 53** → **Hosted zones** → **Create hosted zone**
2. **Domain name**: enter your purchased domain (e.g. `jobtracker-fcj.io.vn`)
3. **Type**: **Public hosted zone**
4. Choose **Create hosted zone**
5. The zone comes with two default records: **NS** and **SOA**. Open the **NS** record and copy all **four nameservers**:

```
ns-1817.awsdns-35.co.uk
ns-968.awsdns-57.net
ns-11.awsdns-01.com
ns-1391.awsdns-45.org
```

![Route 53 hosted zone](/images/5-Workshop/5.9-WAF-Route53/route53-hostedzone.png)

**Cost:** a hosted zone costs **$0.50/month**; DNS query charges are negligible.

#### Step 4: Point nameservers at the registrar

1. Sign in to your registrar's control panel (name.com)
2. Go to **Manage** → select the domain → **Nameservers** tab
3. Choose **Custom**
4. Remove the default nameservers and paste the **four AWS nameservers** (no trailing dot)
5. Choose **Save**

![Name.com nameservers](/images/5-Workshop/5.9-WAF-Route53/name-ns.png)

{{% notice warning %}}
After changing nameservers, DNS needs to **propagate** — anywhere from 15 minutes to a few hours (occasionally 24–48 hours). Track progress at [dnschecker.org](https://dnschecker.org) — select type **NS**; once results return `awsdns...`, propagation is complete.
{{% /notice %}}

#### Step 5: Attach the custom domain to Amplify

1. Go to **Amplify** → your app → **Hosting** → **Custom domains** → **Add domain**
2. Enter the domain (e.g. `jobtrackerfcj.dev`) → **Configure domain**
3. Amplify issues a **free SSL certificate** and displays the DNS records you need to add
4. If the hosted zone is in the same account, Amplify adds the records to Route 53 automatically; otherwise copy them in manually
5. You can map both the root domain and the `www` subdomain
6. Wait for the status to change from **Pending verification** to **Available**

Finally, add the new domain to the **CORS** configuration of API Gateway and the S3 CV bucket (as in section 5.8, Step 7).

**Test:** open `https://<your-domain>` → confirm the HTTPS padlock → sign in → run through the features.

{{% notice tip %}}
Amplify charges **nothing** for custom domains or SSL certificates. Only the Route 53 hosted zone costs $0.50/month if you manage DNS through AWS.
{{% /notice %}}
