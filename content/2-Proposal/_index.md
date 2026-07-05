---
title: "Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Job Tracker Platform using AWS Serverless Architecture
## Job Application Tracking Platform on AWS Serverless Architecture

### 1. Executive Summary

The Job Tracker Platform is a centralized platform that helps students and job seekers optimize the job application process, replacing manual tracking with Excel. The system supports CV management, application status tracking, and automatically sends follow-up reminder emails to employers.

The platform is deployed on an AWS Serverless architecture (React, Amazon S3, CloudFront, Amazon Cognito, API Gateway, AWS Lambda, and Amazon DynamoDB), enabling automatic scalability while optimizing operational costs. It also integrates Amazon EventBridge, Amazon SES, AWS WAF, and Amazon CloudWatch to automate email notifications, enhance security, and provide a solid foundation for future data analytics and AI-driven features.

### 2. Problem Statement

*Current Problem*

Currently, many students and job seekers still manage their job application process manually using tools such as Microsoft Excel, Google Sheets, or note-taking applications. Storing information across multiple platforms makes it difficult to track the status of each application and increases the risk of missing important milestones, such as interview schedules, response deadlines, or follow-up dates with employers.

In addition, application documents such as CVs, cover letters, and related files are often stored in different locations on personal computers or cloud storage services. This makes searching and managing documents time-consuming. Users also lack a tool to track the number of submitted applications, evaluate the effectiveness of their job search, or monitor application progress across different stages.

Furthermore, many existing job application management systems only provide basic features or require paid subscriptions to access advanced functionality. This limits accessibility for students and new job seekers who need a comprehensive yet cost-effective solution. Therefore, there is a need for a centralized, secure, and user-friendly platform that manages the entire job application process while leveraging AWS Serverless services to reduce operational costs, improve scalability, and automate tasks such as reminder emails and application management.

*Solution*

The proposed solution uses Amazon Cognito for user authentication and identity management, ensuring that only authenticated users can access the system. The web interface is developed with React and hosted on Amazon S3, while Amazon CloudFront and Amazon Route 53 provide fast content delivery and domain management. Business logic is handled through Amazon API Gateway and AWS Lambda, eliminating the need to manage servers while reducing operational costs.

Application data is stored in Amazon DynamoDB, while CVs and supporting documents are securely stored in Amazon S3 using a Private Bucket. Amazon EventBridge automatically checks applications that require follow-up actions and triggers AWS Lambda to send reminder emails through Amazon SES, helping users avoid missing important milestones throughout the recruitment process.

In addition, the solution integrates AWS WAF to protect the application from malicious requests, Amazon CloudWatch to monitor system performance and operations, and AWS CloudTrail to record activity logs within the AWS environment. With its Serverless architecture, the platform can scale flexibly based on user demand, optimize operational costs, and provide a foundation for future enhancements such as application analytics, data analysis, and AI integration.

*Benefits and Return on Investment (ROI)*

Deploying the Job Tracker Platform on AWS Serverless architecture reduces infrastructure and operational costs through the pay-as-you-go pricing model while eliminating the need for server management. The platform enables users to manage their job applications in a centralized system, reducing the time required to track applications and minimizing the risk of missing important milestones through automated reminder notifications. In addition, the Serverless architecture provides flexible scalability and establishes a strong foundation for future feature expansion, ensuring long-term value and maximizing return on investment.

### 3. Solution Architecture

*Architecture Objectives*  
The architecture of the Job Tracker Platform is designed based on the AWS Serverless model to build a system that is scalable, secure, cost-efficient, and does not require server management. The system separates the user interface, business logic layer, and data storage layer, making it easier to maintain, upgrade, and extend with new features in the future.

*Architecture Overview*  
* The system architecture consists of three main layers:
* **Frontend Layer:** Developed with React and deployed on Amazon S3, integrated with Amazon CloudFront and Amazon Route 53 to deliver content quickly and efficiently to users.
* **Backend Layer:** Users authenticate through Amazon Cognito. Requests from the frontend are routed to Amazon API Gateway and processed by AWS Lambda functions to execute the application's business logic.
* **Data Layer:** Application data is stored in Amazon DynamoDB, while CVs and supporting documents are stored in Amazon S3. Amazon EventBridge and Amazon SES are used to automatically send follow-up reminder emails, while Amazon CloudWatch and AWS CloudTrail monitor system performance and record operational activities.

![Job Tracker Architecture](/images/sd.png)


*AWS Services Used*  
- *Amazon Route 53*: Manages the domain name and routes users to the system.
- *AWS WAF*: Protects the application against malicious requests and common web attacks.
- *Amazon CloudFront*: Delivers content with low latency and high transfer speeds.
- *Amazon S3*: Stores the React frontend and users' CV files.
- *Amazon Cognito*: Authenticates users and manages access control.
- *Amazon API Gateway*: Receives and routes API requests to AWS Lambda.
- *AWS Lambda*: Processes the application's business logic and backend functions.
- *Amazon DynamoDB*: Stores user information and job application data.
- *Amazon EventBridge*: Triggers scheduled and automated tasks.
- *Amazon SES*: Sends follow-up reminder emails and notifications to users.
- *Amazon CloudWatch*: Monitors system performance and collects application logs.
- *AWS CloudTrail*: Records AWS activity history and supports security auditing.

*Component Design*  
- *Edge Layer*: Uses Amazon Route 53, AWS WAF, and Amazon CloudFront to route user requests, protect the system from malicious traffic, and accelerate content delivery.
- *Presentation Layer*: The user interface is developed with React and deployed on Amazon S3, allowing users to manage job applications, CVs, and application status through a web browser.
- *User Management*: Amazon Cognito handles user registration, sign-in, and authentication using JWT tokens to control access to the system.
- *Application Layer*: Amazon API Gateway receives user requests and forwards them to AWS Lambda, which processes business functions such as job application management, CV management, and generating pre-signed URLs.
- *Data Layer*: Job application data is stored in Amazon DynamoDB, while CVs and supporting documents are securely stored in Amazon S3.
- *Monitoring and Operations*: Amazon EventBridge, Amazon SES, Amazon CloudWatch, and AWS CloudTrail automate reminder emails, monitor system performance, and record operational logs.

### 4. Technical Implementation

*Implementation Phases*

* Infrastructure Deployment  
The system is deployed on an AWS Serverless architecture, using Amazon Route 53 for domain management, AWS WAF for application protection, Amazon CloudFront for content delivery, and Amazon S3 for hosting the web interface. This architecture provides flexible scalability while reducing operational costs.

* Application Deployment  
The user interface is developed with React and connected to Amazon API Gateway, which forwards requests to AWS Lambda functions. Users authenticate through Amazon Cognito and use JWT tokens to access the system's features.

* Data Deployment  
User information and job application data are stored in Amazon DynamoDB, while CVs are stored in Amazon S3 using a Private Bucket. Separating structured data from file storage improves security, simplifies management, and enhances system performance.

* Monitoring and Security  
The system uses Amazon EventBridge to trigger scheduled tasks, Amazon SES to send reminder emails, Amazon CloudWatch to monitor system performance, and AWS CloudTrail to record operational activities. In addition, AWS WAF protects the application against unauthorized access and common web attacks.

* Technology Stack Summary

| Component | Technology |
|-----------|------------|
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

* Deployment Process

1. Develop the user interface using React.
2. Deploy the frontend to Amazon S3 and distribute content through Amazon CloudFront.
3. Configure the domain using Amazon Route 53.
4. Set up user authentication with Amazon Cognito.
5. Build REST APIs using Amazon API Gateway.
6. Develop AWS Lambda functions to implement the application's business logic.
7. Store application data in Amazon DynamoDB and CV files in Amazon S3.
8. Configure Amazon EventBridge and Amazon SES to send automated reminder emails.
9. Configure Amazon CloudWatch, AWS CloudTrail, and AWS WAF for system monitoring and security.

*Technical Requirements*

| Category | Technical Requirements |
|:----------|:-----------------------|
| **Platform** | Amazon Web Services (AWS) |
| **Frontend** | React, HTML, CSS, JavaScript |
| **Backend** | AWS Lambda, Amazon API Gateway |
| **Database** | Amazon DynamoDB |
| **Storage** | Amazon S3 (Private Bucket) |
| **User Authentication** | Amazon Cognito (JWT) |
| **Networking & Content Delivery** | Amazon Route 53, Amazon CloudFront |
| **Security** | AWS WAF |
| **Automation** | Amazon EventBridge, Amazon SES |
| **System Monitoring** | Amazon CloudWatch, AWS CloudTrail |
| **Development Tools** | Visual Studio Code, Git, GitHub, Node.js |

---
### 5. Implementation Roadmap & Milestones

- *Pre-Internship (Month 0)*: Spend one month gathering requirements, planning the project, and designing the solution architecture.
- *Internship Period (Months 1–3)*:
    - **Month 1**: Learn AWS services and develop the frontend application.
    - **Month 2**: Implement backend services and integrate AWS components.
    - **Month 3**: Deploy, test, optimize, and release the system.
- *Post-Deployment*: Continue monitoring, optimizing, and extending the platform with new features over the next year.

### 6. Cost Estimation
The costs can be viewed above. [AWS Pricing Calculator](https://calculator.aws/#/estimate?id=621f38b12a1ef026842ba2ddfe46ff936ed4ab01)  
Or download [budget estimate file](../attachments/budget_estimation.pdf).  

*Infrastructure Cost*

| AWS Service | Purpose | Estimated Cost (USD/Month) |
|:------------|:--------|---------------------------:|
| Amazon Route 53 | Domain management | $0.50 |
| Domain (.com) | Website access | $1.20 |
| Amazon CloudFront | Content delivery | $1.50 |
| Amazon S3 | Website and CV storage | $1.00 |
| Amazon Cognito | User authentication | $0.00 |
| Amazon API Gateway | API management | $1.00 |
| AWS Lambda | Business logic processing | $0.50 |
| Amazon DynamoDB | Data storage | $1.50 |
| Amazon EventBridge | Task automation | $0.00 |
| Amazon SES | Follow-up reminder emails | $0.50 |
| AWS WAF | Application protection | $8.00 |
| Amazon CloudWatch | System monitoring | $2.00 |
| AWS CloudTrail | Activity logging | $0.00 |

*Total*: **17.70 USD/month**, **212.40 USD/year**

- *Note*: This is an estimated cost for a demonstration environment or educational project. If the number of users or traffic increases, the costs of services such as Amazon CloudFront, AWS Lambda, Amazon API Gateway, Amazon DynamoDB, and Amazon S3 will increase based on actual usage. AWS follows a **Pay-as-you-go** pricing model, meaning users only pay for the resources they consume.

### 7. Risk Assessment

*Security Risks*  
The system may be exposed to unauthorized access or security vulnerabilities if authentication mechanisms are not properly configured. This could result in unauthorized access to user data or CV files. To mitigate these risks, the platform uses Amazon Cognito for user authentication, JWT tokens for access control, and AWS WAF to protect against common web attacks.

*Performance Risks*  
When the number of users or requests increases significantly, the system may experience longer response times or processing errors. This can negatively impact the user experience. The AWS Serverless architecture enables services such as AWS Lambda and Amazon DynamoDB to automatically scale according to workload demands.

*Data Storage Risks*  
Job application data and CV files are valuable assets that require secure storage. Configuration errors or incorrect access permissions may lead to data loss or unauthorized access. The platform stores files in Amazon S3 Private Buckets and application data in Amazon DynamoDB while leveraging AWS access control mechanisms to ensure data security and integrity.

*Cost Risks*  
Since AWS uses a pay-as-you-go pricing model, operational costs may increase as the number of users and requests grows. Without proper monitoring, expenses may exceed the planned budget. To address this risk, AWS Billing and Amazon CloudWatch are used to monitor resource usage and optimize infrastructure costs.

*Operational Risks*  
Configuration errors, software bugs, or service interruptions may occur during deployment or system updates. These issues can temporarily affect system availability and user experience. To minimize operational risks, the platform is thoroughly tested before deployment, while Amazon CloudWatch and AWS CloudTrail are used for monitoring, logging, and troubleshooting.

### 8. Expected Outcomes

*System Completion*: Successfully develop the Job Tracker Platform on an AWS Serverless architecture, providing a centralized solution for managing the job application process. The platform will support job application management, CV storage, application status tracking, and automated reminder emails.

*Enhanced User Experience*: Users will be able to easily track their application progress and manage documents through an intuitive interface. Automated reminders help reduce the risk of missing important recruitment milestones.

*Performance and Security*: The platform will provide flexible scalability through AWS Serverless architecture while ensuring data security with services such as Amazon Cognito, AWS WAF, and Amazon CloudWatch. This enables the system to operate reliably and securely.

*Future Scalability*: The architecture is designed to support future expansion, allowing new features such as application analytics, data analysis, and AI integration to be added without requiring major architectural changes.