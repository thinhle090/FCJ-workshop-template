---
title : "Introduction"
date : "2026-07-07"
weight : 1
chapter : false
pre : " <b> 5.1 </b> "
---

#### Serverless architecture

- **Serverless** is a model for building applications where developers never manage servers. AWS provisions, scales and patches the underlying infrastructure.
- Serverless services such as **AWS Lambda**, **Amazon DynamoDB**, **Amazon S3** and **Amazon API Gateway** bill by actual usage — there is no charge for idle resources.
- **Amazon Cognito** handles user authentication and issues **JWT tokens**, letting API Gateway verify the caller's identity without any custom auth logic.

#### The problem

Job seekers often submit dozens of applications at once, easily forget when to follow up with recruiters, and struggle to track which CV was sent to which company.

**The solution:** a web application that centralizes all job applications, attaches a dedicated CV to each one, and automatically emails a reminder when a follow-up is due.

#### Workshop overview

In this workshop you will build a system with two parts:

- **Frontend** — a React application (Vite + TypeScript + Tailwind) built and delivered through **AWS Amplify Hosting** with a global CDN, automatic HTTPS and CI/CD from GitHub. **AWS WAF** filters malicious traffic at the edge, and **Amazon Route 53** manages DNS for the custom domain.

- **Backend** — entirely serverless in the **ap-southeast-1 (Singapore)** region: Cognito authenticates users, API Gateway (HTTP API) serves as the entry point with a JWT authorizer, four Lambda functions handle the logic, DynamoDB stores job data, and S3 with KMS stores encrypted CVs. EventBridge triggers a Lambda each morning to scan for due jobs and send emails through SES, with failures routed to an SQS Dead Letter Queue. A CloudWatch Alarm notifies the admin via SNS, and CloudTrail writes audit logs to S3.

![System architecture](/images/5-Workshop/5.1-Introduction/architecture.png)

#### Application flows

| # | Flow | Services |
|---|------|----------|
| 1 | Domain access, edge filtering of malicious traffic | Route 53, AWS WAF |
| 2 | Serve the React UI over CDN + HTTPS | AWS Amplify Hosting |
| 3 | Sign in, receive a JWT | Amazon Cognito |
| 4 | Validate the token, route the request | API Gateway (HTTP API) |
| 5 | Business logic | AWS Lambda |
| 6 | Job data storage | Amazon DynamoDB (+GSI) |
| 7 | CV storage via Presigned URLs | Amazon S3 + AWS KMS |
| 8 | Daily automatic reminders | EventBridge + SES + SQS DLQ |
| 9 | Monitoring and admin alerts | CloudWatch Alarm + SNS |
| 10 | Audit logging | AWS CloudTrail + S3 |

#### Design principles

{{% notice info %}}
**No VPC** — a purely serverless architecture where every component is a managed service protected by IAM, WAF and KMS. This reduces complexity and eliminates NAT Gateway costs (~$32/month).

**No DynamoDB Scans** — every query uses the `userId` partition key or a GSI, keeping performance and cost low as data grows.

**Strict data isolation** — `userId` always comes from the JWT validated by API Gateway; client-supplied data is never trusted.
{{% /notice %}}

Actual cost across the entire development process: **$0.85** (details in section 5.12).
