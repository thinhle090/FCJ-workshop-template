---
title: "Blog 2"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# [SECURITY/Architecture] SECURE ACCESS CONTROL FOR MULTI-USER RAG APPLICATIONS WITH AMAZON BEDROCK AND VERIFIED PERMISSIONS

### Introduction

During my research on Generative AI architectures deployed on Amazon Web Services (AWS), I came across an AWS Architecture Blog article describing the **Secure Access Control for Multi-user RAG Applications** architecture. The article focuses on building a centralized authorization mechanism for Retrieval-Augmented Generation (RAG) applications, enabling user-specific data access control while maintaining a shared Knowledge Base.

This article summarizes and analyzes the architecture proposed by AWS, highlighting the role of each component in building secure, scalable, and manageable multi-user RAG systems based on the **Security by Design** principle.

### Background

RAG applications are increasingly being adopted by enterprises to leverage internal knowledge through Large Language Models (LLMs).

However, the most critical challenge is not data retrieval itself but **access control**.

For example:

- HR employees should only access HR documents.
- Sales employees should only access Sales documents.
- Executives may require access to multiple categories of documents.

A common implementation approach is to create separate Knowledge Bases for each department. However, this approach increases:

- Storage costs.
- Operational costs.
- Management complexity.
- System scalability challenges.

AWS proposes an alternative approach by using **a single shared Knowledge Base** combined with dynamic authorization through **Amazon Verified Permissions**.

### 1. API-Level Access Control

#### Problem

If every request is forwarded directly to the backend or Knowledge Base, the system cannot effectively prevent unauthorized access at the entry point.

This increases the risk of:

- Unauthorized API access.
- API reconnaissance attacks.
- Additional processing load on backend services.

#### Analysis

AWS implements the first security layer at **Amazon API Gateway**.

Each request is:

- Authenticated using Amazon Cognito.
- Forwarded to a Lambda Authorizer.
- The Lambda Authorizer sends an authorization request to Amazon Verified Permissions.

If the user is not authorized, the request is rejected before reaching the application backend.

#### Benefits

- Blocks unauthorized access at the API Gateway.
- Reduces backend workload.
- Enforces the Principle of Least Privilege.
- Reduces the overall attack surface.

### 2. Document-Level Access Control

#### Problem

Even when a user is authorized to access the API, they should not necessarily have access to every document stored within the Knowledge Base.

If the LLM retrieves information from the entire Knowledge Base, sensitive data may be exposed across departments.

#### Analysis

AWS introduces a second layer of protection through a **Middleware Lambda**.

The middleware sends another authorization request to Amazon Verified Permissions to determine:

- Which group the user belongs to.
- Which document categories the user is permitted to access.
- Which Metadata Filter should be generated.

The generated Metadata Filter is then passed directly to Amazon Bedrock's **RetrieveAndGenerate** API.

As a result, the language model retrieves only documents that the user is authorized to access.

### Figure 1. Multi-user RAG Access Control Architecture

![RAG Architecture with Amazon Verified Permissions](/images/h1.2.jpg)

**Figure 1** illustrates a multi-layer access control architecture for RAG applications on AWS.

The workflow consists of the following steps:

1. Users access the application through Amazon CloudFront.
2. CloudFront delivers the web application hosted on Amazon S3.
3. Amazon Cognito authenticates users and issues a JWT token.
4. Requests pass through AWS WAF for web-layer threat inspection.
5. Amazon API Gateway receives the request and forwards it to the Lambda Authorizer.
6. The Lambda Authorizer requests an authorization decision from Amazon Verified Permissions.
7. Amazon Verified Permissions evaluates Cedar policies and returns an Allow or Deny decision.
8. If access is granted, API Gateway forwards the request to the Middleware Lambda.
9. The Middleware Lambda queries Amazon Verified Permissions again to generate an appropriate Metadata Filter.
10. The middleware invokes Amazon Bedrock Knowledge Bases using the generated Metadata Filter.
11. Amazon Bedrock retrieves information only from the filtered document set stored in Amazon S3 Vectors before generating a response.

This architecture establishes two independent security layers, reducing unauthorized access and preventing data leakage across different user groups.

### 3. Centralized Policy Management with Amazon Verified Permissions

#### Problem

When authorization logic is embedded directly into application code, every policy change requires code modifications and application redeployment.

This increases:

- Maintenance costs.
- The possibility of implementation errors.
- Management complexity.

#### Analysis

Amazon Verified Permissions uses the **Cedar** policy language to define authorization rules.

All authorization policies are centrally managed and completely separated from application source code.

Whenever changes occur involving:

- Users.
- Roles.
- Departments.
- Access policies.

Administrators only need to update the authorization policies without modifying or redeploying the application.

#### Benefits

- Centralized policy management.
- Easy scalability.
- Simplified auditing.
- Compliance with the Deny-by-default principle.

### 4. Defense-in-Depth Architecture for GenAI Applications

#### Analysis

The AWS architecture follows the **Defense in Depth** model, where each security layer performs an independent protection function.

The security layers include:

- Amazon Cognito for user authentication.
- AWS WAF for web application protection.
- Amazon API Gateway as the API entry point.
- Lambda Authorizer for authorization validation.
- Amazon Verified Permissions for policy evaluation.
- Middleware Lambda for Metadata Filter generation.
- Amazon Bedrock for authorized document retrieval only.

Performing authorization checks across multiple layers reduces the likelihood of configuration errors and significantly minimizes the risk of unauthorized data exposure.

### Architecture Summary

| Component | AWS Service | Role |
|------------|-------------|------|
| Authentication | Amazon Cognito | User authentication |
| Edge Protection | AWS WAF | Web application protection |
| API Layer | Amazon API Gateway | API request management |
| Authorization | Amazon Verified Permissions | Authorization evaluation |
| Policy Engine | Cedar | Authorization policy management |
| AI Platform | Amazon Bedrock Knowledge Bases | Knowledge retrieval for RAG |
| Data Storage | Amazon S3 Vectors | Vector data storage |

### Conclusion

The architecture proposed by AWS demonstrates that building secure Generative AI applications depends not only on the language model itself but also on implementing effective access control mechanisms.

By integrating Amazon Cognito, AWS WAF, Amazon API Gateway, Amazon Verified Permissions, and Amazon Bedrock, organizations can establish a multi-layer security architecture that minimizes unauthorized access, prevents cross-department data leakage, and improves system scalability.

This approach provides an effective foundation for deploying secure multi-user RAG applications following the **Security by Design** and **Defense in Depth** principles.

### References

https://aws.amazon.com/vi/blogs/architecture/secure-multi-tenant-rag-with-amazon-bedrock-and-verified-permissions/

### Article Link

https://www.facebook.com/groups/awsstudygroupfcj/permalink/2202713613826932/?rdid=moGmZYTchQTj5BBA#