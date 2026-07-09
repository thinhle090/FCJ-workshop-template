---
title: "Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Job Tracker Platform using AWS Serverless Architecture
## Job Application Process Management Platform on AWS Serverless Architecture

### 1. Operational Summary
The Job Tracker Platform supports students and job seekers in managing their entire application process in a centralized system, including job management, CV storage, application status tracking, and automated follow-up email reminders.

The system is built on AWS Serverless architecture, utilizing AWS Amplify Hosting, Amazon Cognito, Amazon API Gateway, AWS Lambda, Amazon DynamoDB, and Amazon S3 to ensure scalability, security, and optimized operating costs. In addition, the system integrates AWS WAF, AWS KMS, Amazon EventBridge Scheduler, Amazon SES, Amazon CloudWatch, and AWS CloudTrail to enhance security, automation, and monitoring, while also providing a foundation for future expansion of data analytics and AI capabilities.

### 2. Problem Statement
*Current Problem*
Currently, many students and job seekers still manage their application processes using manual tools such as Microsoft Excel, Google Sheets, or notes on their phones. Each application typically includes a lot of information such as company name, job title, processing status, interview schedule, deadline for sending follow-up emails, and attached documents such as CVs or cover letters. As the number of applications increases, managing them using these methods becomes difficult, prone to errors, and time-consuming.

In addition, CVs and documents are often stored in various locations on the computer or on cloud storage services, making it difficult for users to search and manage them centrally. The lack of proper access control and data protection mechanisms also increases the risk of unauthorized access or disclosure of personal information.

Furthermore, users often have to remember when to contact employers after submitting their applications. Missing important deadlines such as interview schedules or follow-up appointments can negatively impact job opportunities. Existing recruitment management platforms are primarily geared towards businesses or require paid access to full features, while there are few suitable solutions for students and individual job seekers.

Technically, many traditional web applications require the deployment and management of servers, databases, and network infrastructure, resulting in high operating costs, difficulty scaling up as the number of users increases, and significant effort in monitoring, maintaining, and ensuring system security.

*Solution*
To address the above issues, this project builds a Job Tracker Platform based on AWS Serverless architecture, managing the entire application process on a centralized, secure, and scalable platform.

The user interface is developed using React and deployed on AWS Amplify Hosting, enabling automatic deployment (CI/CD) from GitHub and content distribution via a global CDN network with default HTTPS certificates. Users access the system via a private domain managed by Amazon Route 53, while AWS WAF protects the application against common attacks such as SQL Injection, Cross-Site Scripting (XSS), malicious bots, and high-risk IP addresses.

The system uses Amazon Cognito to manage registration, login, and user authentication using JWT tokens. After successful authentication, requests are sent to the Amazon API Gateway, where the JWT Authorizer verifies the token's validity before forwarding it to the AWS Lambda functions for business process handling. Key functions include managing job applications, tracking application status, storing recruitment information, and generating presigned URLs for uploading or downloading CV files.

Application information is stored in Amazon DynamoDB, where all data queries are based on the user ID obtained from the JWT, ensuring each user can only access their own data. CV files and attachments are stored in Amazon S3 Private Buckets, accessible only through presigned URLs with limited validity, enhancing security and reducing system load.

To help users avoid missing important milestones in the recruitment process, the Amazon EventBridge Scheduler is configured to run daily, triggering Lambda to check applications requiring follow-up. When the scheduled time arrives, the system automatically sends reminder emails via Amazon SES and updates the status to avoid duplicate submissions. In case of errors, the Retry mechanism and Amazon SQS Dead Letter Queue (DLQ) help save unfinished tasks for later troubleshooting.

In addition, the system uses Amazon CloudWatch to monitor logs, metrics, and set up alerts when errors are detected, while AWS CloudTrail records the entire history of operations on AWS accounts to support auditing and investigation in case of security incidents.

*Benefits and Return on Investment (ROI)* Deploying the Job Tracker Platform on the AWS Serverless platform offers many benefits in terms of both technical and operational costs. Thanks to the Pay-as-you-go model, the system only incurs costs when resources are actually used, and there is no need to invest in or manage physical servers. In testing and learning project environments, most services such as AWS Lambda, Amazon API Gateway, Amazon DynamoDB, Amazon S3, Amazon Cognito, Amazon SES, Amazon SNS, Amazon SQS, Amazon CloudWatch, and AWS CloudTrail are in the AWS Free Tier, significantly reducing deployment costs. The main costs come from AWS WAF and Amazon Route 53, with an estimated total cost of around $24/month, but this is offset by AWS Free Tier Credits, resulting in virtually zero actual development costs.

For users, the system allows for centralized management of the entire application process on a single interface, supporting resume storage, application status tracking, and automated follow-up email reminders. This reduces manual management time, minimizes missed job opportunities, and improves job search efficiency.

Technically, the Serverless architecture allows the system to automatically scale with the number of users without changing the infrastructure. Simultaneously, the integration of Amazon Cognito, AWS WAF, Presigned URL, Amazon CloudWatch, and AWS CloudTrail enhances security, monitoring, and auditing capabilities. This architecture also provides a favorable foundation for expanding future features such as application data analysis, job search performance statistics, artificial intelligence (AI) integration, and machine learning services without changing the overall architecture.
### 3. Solution Architecture
#### Architectural Objectives

The **Job Tracker Platform** architecture is built on the **AWS Serverless Architecture** model to develop a scalable, flexible job application management system that ensures high availability, enhanced security, and optimized operating costs. The entire system is deployed on AWS Managed Services, eliminating the need for server management and automatically scaling according to actual traffic.

The architecture is designed to separate the user interface, business process, data storage, and system monitoring layers. Simultaneously, authentication, authorization, data encryption, and activity monitoring mechanisms are integrated from the outset according to the **Security by Design** principle, contributing to the security and reliability of the entire system.

#### Specific Architecture

The system is built using a multi-tier architecture, including the following main components:

- **Edge Layer:** Users access the system through a domain managed by **Amazon Route 53**. All requests are checked by **AWS WAF** to prevent common attacks such as SQL Injection (SQLi), Cross-site Scripting (XSS), malicious bots, and high-risk IP addresses before being forwarded to the application.

- **Frontend Layer:** The user interface is developed using **React** and deployed on **AWS Amplify Hosting**. Amplify automatically deploys the application from GitHub via the CI/CD process and distributes content through a global CDN network with automatically configured HTTPS.

- **Authentication Layer:** Users log in via **Amazon Cognito**. After successful authentication, Cognito issues a **JSON Web Token (JWT)** to verify identity and access in subsequent requests.

- **Backend:** Application requests are sent to the **Amazon API Gateway (HTTP API)**. The JWT Authorizer authenticates the token before forwarding the request to the corresponding **AWS Lambda** functions to handle functions such as job management, data querying, and presigned URL generation.

- **Data Layer:** Application information is stored on **Amazon DynamoDB**, while CV and document files are stored in the **Amazon S3 Private Bucket**. All data is encrypted using **AWS KMS** to ensure security.

- **Automation Layer:** The **Amazon EventBridge Scheduler** triggers Lambda on a schedule to check profiles up to the follow-up point. Lambda sends reminder emails via **Amazon SES**, updates the sending status, and uses **Amazon SQS Dead Letter Queue (DLQ)** to store failed requests after multiple retries.

- **Monitoring & Auditing Layer**: **Amazon CloudWatch** collects system activity logs and metrics, and sends alerts via **Amazon SNS** when errors are detected. **AWS CloudTrail** records all API calls and stores logs in a separate S3 Bucket for auditing purposes.

![Job Tracker Architecture](/images/sdt2.png)

#### AWS Services Used

| AWS Service | Role |
|-------------|------|
| **Amazon Route 53** | Manages DNS resolution and routes users to the application |
| **AWS WAF** | Protects the application against SQL injection, XSS, bots, and malicious requests |
| **AWS Amplify Hosting** | Hosts and delivers the React application with built-in CI/CD support |
| **Amazon Cognito** | Handles user registration, authentication, and JWT-based authorization |
| **Amazon API Gateway (HTTP API)** | Receives and routes API requests to backend services |
| **AWS Lambda** | Executes all business logic of the application |
| **Amazon DynamoDB** | Stores user information and job application data |
| **Amazon S3** | Stores the static web application and CV files in a private bucket |
| **AWS KMS** | Encrypts data stored in Amazon DynamoDB and Amazon S3 |
| **Amazon EventBridge Scheduler** | Automatically triggers scheduled tasks |
| **Amazon SES** | Sends automated follow-up reminder emails |
| **Amazon SQS (DLQ)** | Stores failed email processing tasks for later retry |
| **Amazon CloudWatch** | Monitors application performance, logs, and metrics |
| **Amazon SNS** | Sends alert notifications to administrators |
| **AWS CloudTrail** | Records AWS API activities for auditing and compliance |

#### Component Design

- **Edge Layer:** Includes **Amazon Route 53**, **AWS WAF**, and **AWS Amplify Hosting**, responsible for user routing, protecting the application from common attacks, and distributing the React interface via a global CDN.

- **Frontend Layer:** The React application is deployed on AWS Amplify, allowing users to manage the application process, update job status, upload or download CVs, and track follow-up schedules through a web browser.

- **Authentication Layer:** Amazon Cognito performs registration, login, and JWT Token issuance. The API Gateway uses JWT Authorizer to authenticate users before allowing access to APIs.

- **Application Layer:** **AWS Lambda** handles all business processes such as task management, data retrieval, creating Presigned URLs for S3, and processing scheduled email reminders.

- **Data Layer:** **Amazon DynamoDB** stores user information and application data, while **Amazon S3 Private Bucket** stores CV files and documents. All data is encrypted using AWS KMS for security.

- **Automation Layer:** **Amazon EventBridge Scheduler** triggers Lambda on a schedule to check for profiles requiring follow-up. Lambda sends emails via Amazon SES and uses Amazon SQS Dead Letter Queue to store failed requests to prevent data loss.

- **Monitoring & Auditing Layer:** Amazon CloudWatch collects logs, metrics, and generates alerts via Amazon SNS when system errors are detected. AWS CloudTrail records all AWS account activity and stores logs for auditing and incident investigation.

### 4. Technical Deployment
*Deployment Phases*
#### Infrastructure Deployment

The system is deployed on an **AWS Serverless** platform to reduce operating costs and eliminate server management. The domain is configured via **Amazon Route 53**, the user interface is deployed on **AWS Amplify Hosting**, while **AWS WAF** is integrated to protect the application against common attacks such as SQL Injection (SQLi), Cross-site Scripting (XSS), and malicious internet access.

#### Application Deployment

The user interface is developed using **React** and source code is managed on GitHub. AWS Amplify automatically builds and deploys the application whenever there are changes to the source code repository. Users log in via **Amazon Cognito**, receive a **JWT Token**, and use this token to access APIs provided by the **Amazon API Gateway**.

#### Backend Implementation

Business functions are built as **AWS Lambda** functions and routed through the **Amazon API Gateway (HTTP API)**. The JWT Authorizer authenticates the user before forwarding the request to the corresponding Lambda to process functions such as job management, data querying, and generating Presigned URLs for uploading or downloading CV files.

#### Data Implementation

Information about the application process is stored on **Amazon DynamoDB**, while CV files are stored in the **Amazon S3 Private Bucket**. File upload and download operations use **Presigned URLs** to allow users to directly access S3 without passing data through Lambda. Data on DynamoDB and S3 is encrypted using **AWS KMS** to ensure information security.

#### Automation Implementation

The system uses **Amazon EventBridge Scheduler** to automatically trigger Lambda according to a schedule to check records up to the follow-up point. Lambda sends reminder emails via **Amazon SES**, then updates the status to avoid repeated sending. If the email fails after multiple attempts, the message is forwarded to the **Amazon SQS Dead Letter Queue (DLQ)** for processing.

#### Monitoring and Security

All system activity is monitored via **Amazon CloudWatch**, including logs, metrics, and alarms. When the number of errors exceeds the configured threshold, CloudWatch sends notifications to administrators via **Amazon SNS**. Simultaneously, **AWS CloudTrail** records all API calls generated within the AWS account and stores logs on Amazon S3 for auditing and incident investigation purposes.

* Summary of technologies used
| Component | Technology / Service |
|-----------|----------------------|
| Frontend | React, HTML, CSS, JavaScript |
| Source Control | Git, GitHub |
| Hosting | AWS Amplify Hosting |
| Domain & DNS | Amazon Route 53 |
| Authentication | Amazon Cognito (JWT) |
| API | Amazon API Gateway (HTTP API) |
| Business Logic | AWS Lambda |
| Database | Amazon DynamoDB |
| File Storage | Amazon S3 (Private Bucket) |
| File Security | Amazon S3 Pre-signed URL |
| Encryption | AWS Key Management Service (AWS KMS) |
| Scheduler | Amazon EventBridge Scheduler |
| Email Service | Amazon Simple Email Service (Amazon SES) |
| Message Queue | Amazon Simple Queue Service (Amazon SQS) – Dead Letter Queue (DLQ) |
| Notification | Amazon Simple Notification Service (Amazon SNS) |
| Monitoring | Amazon CloudWatch |
| Audit Logging | AWS CloudTrail |
| Security | AWS WAF |

* Deployment Process
1. Develop the user interface using React and manage the source code on GitHub.

2. Deploy the application to AWS Amplify Hosting and configure the domain using Amazon Route 53.
3. Integrate AWS WAF to protect the application against internet attacks.

4. Configure Amazon Cognito to manage users and authenticate using JWT Tokens.

5. Build APIs on Amazon API Gateway (HTTP API).

6. Develop AWS Lambda functions to handle system business logic.

7. Store application data on Amazon DynamoDB and CV files on Amazon S3 Private Bucket.

8. Use Presigned URLs for users to upload and download files directly from Amazon S3.

9. Set up Amazon EventBridge Scheduler to automatically check profiles for follow-up and send emails via Amazon SES.

10. Configure Amazon SQS Dead Letter Queue to save failed email delivery requests.

11. Set up Amazon CloudWatch, Amazon SNS, and AWS CloudTrail to monitor, send alerts, and log system activity.

*Technical Requirements*
| Category | Technical Requirement |
|:---------|:----------------------|
| **Cloud Platform** | Amazon Web Services (AWS) |
| **Frontend** | React, HTML5, CSS3, JavaScript |
| **Source Control** | Git, GitHub |
| **Hosting** | AWS Amplify Hosting |
| **Domain & DNS** | Amazon Route 53 |
| **User Authentication** | Amazon Cognito (JWT Authentication) |
| **Backend API** | Amazon API Gateway (HTTP API) |
| **Business Logic** | AWS Lambda |
| **Database** | Amazon DynamoDB |
| **File Storage** | Amazon S3 (Private Bucket, Presigned URL) |
| **Data Encryption** | AWS Key Management Service (AWS KMS) |
| **Task Scheduling** | Amazon EventBridge Scheduler |
| **Email Service** | Amazon Simple Email Service (SES) |
| **Failure Queue** | Amazon SQS Dead Letter Queue (DLQ) |
| **System Monitoring** | Amazon CloudWatch |
| **Alert Notification** | Amazon Simple Notification Service (SNS) |
| **Audit Logging** | AWS CloudTrail |
| **Application Security** | AWS WAF |
| **Development Environment** | Visual Studio Code, Node.js, npm |

### 5. Roadmap & Deployment Milestones

- **Preparation Phase (Month 0):**

- Survey user needs and analyze the application process management problem.

- Research AWS Serverless services suitable for the system requirements.

- Design the overall architecture, data model, and system workflow.

- **Development Phase (Month 1):**

- Build the user interface using React.

- Deploy the application on AWS Amplify Hosting and configure the domain with Amazon Route 53.

- Set up Amazon Cognito to manage users and authenticate using JWT.

- **Backend Construction Phase (Month 2):**

- Build APIs using Amazon API Gateway and AWS Lambda.

- Design the database on Amazon DynamoDB.

- Deploy CV file storage on Amazon S3 using Presigned URLs.

- Integrate AWS KMS for data encryption.

- **System Completion Phase (March):**

- Integrate Amazon EventBridge Scheduler and Amazon SES to send automated email reminders.

- Set up Amazon CloudWatch, Amazon SNS, and AWS CloudTrail for monitoring and logging.

- Configure AWS WAF to enhance application security.

- Test functionality, optimize performance, and deploy the final version.

- **Operation and Expansion Phase:**

- Monitor system activity through CloudWatch.

- Optimize costs and performance based on actual usage.

- Develop additional features such as application process statistics, data analysis, and AI integration to support users in the future.

### 6. Budget Estimate
You can view the cost on the [AWS Pricing Calculator](https://calculator.aws/#/estimate?id=621f38b12a1ef026842ba2ddfe46ff936ed4ab01)

Or download the [budget estimate file](../attachments/budget_estimation.pdf).

*Infrastructure Costs*
## *Infrastructure Cost*

| AWS Service | Purpose | Estimated Cost (USD/Month) |
|:------------|:--------|---------------------------:|
| **AWS Amplify Hosting** | Host and deploy the React application | **$0.00** *(Free Tier)* |
| **Amazon Route 53 Hosted Zone** | DNS management | **$0.50** |
| **AWS WAF** | Protect the web application from common attacks | **~$23.00** |
| **Amazon Cognito** | User authentication | **$0.00** *(Free Tier)* |
| **Amazon API Gateway (HTTP API)** | API management | Included |
| **AWS Lambda** | Business logic processing | Included |
| **Amazon DynamoDB** | Application data storage | Included |
| **Amazon S3** | CV and file storage | Included |
| **AWS Key Management Service (AWS KMS)** | Data encryption | Included |
| **Amazon EventBridge Scheduler** | Scheduled task execution | Included |
| **Amazon Simple Email Service (SES)** | Email notifications | Included |
| **Amazon Simple Notification Service (SNS)** | Alert notifications | Included |
| **Amazon Simple Queue Service (SQS)** | Dead Letter Queue (DLQ) | Included |
| **Amazon CloudWatch** | Monitoring and logging | Included |
| **AWS CloudTrail** | Audit logging | Included |

### *Estimated Monthly Cost*

| Item | Cost |
|:-----|-----:|
| **AWS Backend** (Lambda, API Gateway, DynamoDB, S3, Cognito, SES, SNS, SQS, CloudWatch, CloudTrail, KMS) | **~$0.85/month** |
| **AWS Amplify Hosting** | **$0.00** *(Free Tier)* |
| **Amazon Route 53 Hosted Zone** | **$0.50/month** |
| **AWS WAF** | **~$23.00/month** |
| **Total Estimated Cost** | **~$24.35/month** |

**Total:** **USD 24.35/month** (**USD 292.20/year**)

> **Note:** These estimates are based on a demo or educational environment. As the number of users and request volume increase, services such as AWS Amplify Hosting, API Gateway, AWS Lambda, Amazon DynamoDB, and Amazon S3 may incur additional charges based on actual usage. AWS follows a **Pay-as-you-go** pricing model, meaning you only pay for the resources you consume.

### 7. Risk Assessment
#### Security Risks

The system is at risk of unauthorized access if authentication or authorization mechanisms are incorrectly configured. APIs can become targets of attacks such as SQL Injection, Cross-site Scripting (XSS), Bots, or authentication vulnerability exploits. Additionally, CV data and application information are sensitive data that needs to be protected from leakage.

To mitigate risks, the system uses Amazon Cognito to authenticate users using JWTs, the JWT Authorizer API Gateway to check tokens before forwarding requests to Lambda, AWS WAF to block common web-layer attacks, and S3 Private Bucket and AWS KMS to encrypt stored data.

#### Performance Risks

When the number of users or requests spikes, the system may experience processing delays or an increase in errors if resources are not scaled appropriately.

AWS Serverless architecture mitigates this risk through the automatic scaling capabilities of AWS Lambda, Amazon API Gateway, and Amazon DynamoDB. Additionally, AWS Amplify Hosting combined with a global CDN reduces access latency for users in various regions.

#### Data Storage Risks

Application data and CV files are critical user assets. If storage configurations or permissions are inadequate, data may be accessed illegally or its integrity compromised.

To ensure security, all CVs are stored in Amazon S3 Private Buckets and can only be accessed via short-term presigned URLs. Data in Amazon DynamoDB and Amazon S3 is encrypted using AWS KMS to ensure security during storage.

#### Cost Risks

Although the serverless architecture optimizes costs, increased user numbers or traffic can still lead to higher-than-expected costs, especially for services like AWS WAF, API Gateway, or Lambda.

To control the budget, the system uses Amazon CloudWatch to monitor resource usage and leverages AWS Free Tier and Billing Dashboard to monitor costs and optimize resources during operation.

#### Operational Risks

During system deployment or updates, application errors, configuration errors, or service issues may occur, affecting user experience.

To improve operational efficiency, the system uses Amazon CloudWatch Logs and CloudWatch Alarms to monitor errors in real time. When the number of API errors exceeds the configured threshold, CloudWatch will send alerts via Amazon SNS to administrators. Additionally, EventBridge Scheduler is configured with Retry and SQS Dead Letter Queue (DLQ) to handle failed email delivery tasks, while AWS CloudTrail logs all API activity for troubleshooting and remediation.

### 8. Expected Results
#### System Completion

Successfully built the Job Tracker Platform on AWS Serverless architecture, fully meeting the functionalities such as managing the application process, storing CVs, tracking application status, securely uploading and downloading documents, and supporting automatic email reminders.

#### Enhanced User Experience

Users can manage the entire application process on a single platform with an intuitive interface. The system supports job status updates, centralized record keeping, and automatic follow-up email reminders, helping to minimize missed milestones in the recruitment process.

#### Performance and Security Assurance

The system operates on a Serverless architecture that automatically scales as needed and applies multiple layers of protection such as AWS WAF, Amazon Cognito, JWT Authorizer, S3 Private Bucket, AWS KMS, and CloudTrail to ensure security, data integrity, and system monitoring.

#### Enhanced Operation and Monitoring

All system activity is monitored through Amazon CloudWatch, CloudTrail, and SNS Alert, enabling early detection of issues, sending alerts to administrators, and supporting rapid troubleshooting and remediation.

#### Future Scalability

The architecture is designed to separate the interface, business logic layer, and data layer, making it easy to expand with new functionalities such as application process statistics, data analysis, AI integration to optimize CVs, job suggestions, or building a user behavior analysis system without changing the overall architecture.