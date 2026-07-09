---
title : "Cost"
date : "2026-07-07"
weight : 12
chapter : false
pre : " <b> 5.12 </b> "
---

#### Content
- [Actual cost](#actual-cost)
- [Service-by-service breakdown](#service-by-service-breakdown)
- [Estimate at production scale](#estimate-at-production-scale)
- [Cost optimization decisions](#cost-optimization-decisions)

#### Actual cost

After the entire build and testing process (thousands of API requests, dozens of Lambda deployments, CV uploads, reminder emails):

| Item | Cost |
|---|---|
| Backend (Lambda, API Gateway, DynamoDB, S3, Cognito, SES, SNS, SQS, CloudWatch, CloudTrail, KMS) | **$0.85** |
| Amplify Hosting | **$0** (within free tier) |
| Domain (bought outside AWS) | **$0** |
| Route 53 hosted zone | **$0.50/month** |
| AWS WAF on Amplify | **~$23/month** |
| **Covered by Free Tier credits** | Everything |
| **Actually paid** | **$0.00** |

![Billing](/images/5-Workshop/5.12-Cost/billing.png)

![Credits](/images/5-Workshop/5.12-Cost/credits.png)

#### Service-by-service breakdown

| Service | Free Tier | Notes |
|---|---|---|
| **Lambda** | 1M requests + 400,000 GB-seconds/month | arm64 is ~20% cheaper than x86 |
| **API Gateway (HTTP API)** | 1M requests/month (first 12 months) | ~70% cheaper than REST API |
| **DynamoDB** | 25 GB storage, On-demand billed per request | No Scans → low read cost |
| **S3** | 5 GB storage, 20,000 GET, 2,000 PUT | Bucket Key cuts KMS calls ~90% |
| **Cognito** | 10,000 MAU free | No charges at demo scale |
| **Amplify Hosting** | 1,000 build minutes, 15 GB transfer, 5 GB storage/month | A few-MB static app fits easily |
| **SES** | 200 emails/day when sent from Lambda | Sandbox mode is enough for a demo |
| **CloudWatch** | 10 alarms, 5 GB logs/month | The 5xx alarm stays within limits |
| **CloudTrail** | Management events are free | Only S3 storage costs (a few KB/day) |
| **Route 53** | No free tier | $0.50/month per hosted zone |
| **AWS WAF (Amplify)** | No free tier | See the breakdown below |

#### Breaking down the AWS WAF cost

Many articles quote only **$15/month** — that is the **Amplify fee for the firewall feature**, excluding the actual AWS WAF charges underneath. The full cost has three components:

| Component | Billed by | Cost |
|---|---|---|
| Amplify Firewall (integration fee) | AWS Amplify | **$15/month** |
| WebACL ($5) + 3 managed rules (3 × $1) | AWS WAF | **~$8/month** |
| Request processing | AWS WAF | **$1.40 per 1M requests** |
| **Total** | | **~$23–24/month** |

When you enable the firewall, Amplify creates a WebACL named `CreatedByAmplify-<app-id>` with 3 managed rules, scoped to **CloudFront (Global)**. Standard WAF pricing is $5/month per WebACL plus $1/month per rule — that is where the ~$8 comes from.

{{% notice tip %}}
This estimate appears directly on Amplify's firewall configuration screen before you click **Add firewall**. Always read the "incurs additional costs" note in the bottom-right corner.
{{% /notice %}}

#### Estimate at production scale

Assuming 1,000 monthly active users:

| Service | Monthly estimate |
|---|---|
| Lambda + API Gateway | ~$1–3 |
| DynamoDB On-demand | ~$2–5 |
| S3 (CV storage) | ~$1–2 |
| Amplify Hosting | ~$0–2 |
| Cognito | $0 (under 10,000 MAU) |
| SES + SNS + CloudWatch + CloudTrail | ~$1 |
| Route 53 | $0.50 |
| AWS WAF | ~$23 |
| **Total** | **~$28–36/month** |

Without WAF: **~$5–13/month**.

{{% notice info %}}
A serverless architecture lets cost **grow linearly with real users** instead of paying a fixed price for servers idling 24/7. A single t3.small EC2 instance costs ~$15/month even with zero traffic.
{{% /notice %}}

#### Cost optimization decisions

| Decision | Rationale |
|---|---|
| **HTTP API** instead of REST API | ~70% cheaper with all the features we need |
| **Lambda arm64** instead of x86 | ~20% cheaper at the same performance |
| **DynamoDB On-demand** | No throughput forecasting, no paying for idle capacity |
| **AWS managed KMS key** (`aws/s3`) instead of a customer managed key | Saves $1/month per key |
| **S3 Bucket Key: Enable** | Cuts KMS API calls by ~90% |
| **No VPC** | Avoids NAT Gateway costs (~$32/month) |
| **Presigned URLs** instead of uploading through Lambda | Reduces Lambda execution time and data transfer |
| **No DynamoDB Scans** | Key-based queries are far cheaper than repeated scans |
| **Domain purchased outside AWS** | Domain registration fees are **not covered by promotional credits** |

{{% notice warning %}}
**AWS WAF is the largest single cost (~$23/month)** in this architecture. If you deployed it purely for learning, **disable WAF as soon as you finish** to preserve your credits.
{{% /notice %}}
