---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# [SECURITY/Architecture] Don't Let the Cloud Become a Blind Spot: Designing AWS Infrastructure to Resist Ransomware

### Introduction

During my research on security architecture in Amazon Web Services (AWS), I came across the article **"Let's Architect! Architecting for Security"** published on the AWS Architecture Blog. Rather than introducing individual AWS services, the article focuses on designing systems following the **Security by Design** approach to reduce the attack surface and strengthen **Defense in Depth**.

This article summarizes and analyzes the key architectural principles presented in the reference while highlighting how each component contributes to mitigating ransomware risks in AWS environments.

### Background

Modern ransomware attacks rarely begin by encrypting data immediately. Instead, attackers typically progress through multiple stages to gain control of the environment before launching the final attack.

A typical attack lifecycle includes:

1. Reconnaissance
2. Network Scanning
3. Privilege Escalation
4. Lateral Movement
5. Data Exfiltration
6. Data Encryption

If an environment is designed with inadequate network architecture or improper access control, attackers can rapidly expand their foothold and significantly increase the impact of the incident.

According to AWS, implementing security architecture from the beginning can interrupt or even prevent this attack chain before severe damage occurs.

### 1. Privilege Management with Temporary Elevated Access

#### Problem

In many environments, administrator accounts or IAM Roles are granted privileged permissions for extended periods. If credentials are compromised, attackers can immediately leverage those permissions to gain control of the environment.

#### Analysis

AWS recommends adopting the **Temporary Elevated Access (Just-In-Time Access)** model.

Under this approach:

- Administrative permissions are granted only when necessary.
- Access is provided for a limited period.
- Permissions are automatically revoked once the predefined time expires.

#### Benefits

- Reduces the risk of privileged credential exposure.
- Limits privilege escalation opportunities.
- Enforces the Principle of Least Privilege.
- Narrows the attack window for adversaries.

### 2. Isolating Network Traffic with Amazon VPC Endpoints

#### Problem

Some architectures deploy Amazon EC2 instances or AWS Lambda functions inside Private Subnets while still relying on NAT Gateway to access AWS services such as Amazon S3 or Amazon DynamoDB.

Although the resources remain in private networks, outbound traffic may still traverse the Internet before reaching AWS services.

This introduces several security risks:

- Establishing Command and Control (C&C) communications.
- Data exfiltration.
- Downloading additional malicious payloads.

#### Analysis

AWS recommends using **VPC Gateway Endpoints** or **Interface Endpoints** to establish direct connectivity with AWS services.

This ensures that traffic remains entirely within the AWS private network without traversing the public Internet.

#### Figure 1. Temporary Elevated Access Architecture

![Temporary Elevated Access Architecture](/images/h1.jpg)

**Figure 1** illustrates the Temporary Elevated Access workflow. Instead of assigning permanent administrative privileges, the system utilizes a **Temporary Elevated Access Broker** to centrally manage privileged access requests.

The workflow consists of four steps:

1. The user is authenticated through an **Identity Provider (IdP)**.
2. The broker verifies the user's identity, role, and requested permissions.
3. The broker may integrate with **Change Management** or **Incident Management** systems to validate the request before granting access.
4. Once approved, the broker grants temporary access to the AWS environment for a predefined duration, after which the permissions are automatically revoked.

This architecture minimizes privileged credential exposure, reduces privilege escalation risks, and reinforces the **Principle of Least Privilege**.

#### Benefits

- Reduces the risk of privileged credential exposure.
- Limits privilege escalation opportunities.
- Enforces the Principle of Least Privilege.
- Narrows the attack window for adversaries.

### 2. Isolating Network Traffic with Amazon VPC Endpoints

#### Problem

Some architectures deploy Amazon EC2 instances or AWS Lambda functions inside Private Subnets while still relying on NAT Gateway to access AWS services such as Amazon S3 or Amazon DynamoDB.

Although the resources remain in private networks, outbound traffic may still traverse the Internet before reaching AWS services.

This introduces several security risks:

- Establishing Command and Control (C&C) communications.
- Data exfiltration.
- Downloading additional malicious payloads.

#### Analysis

AWS recommends using **VPC Gateway Endpoints** or **Interface Endpoints** to establish direct connectivity with AWS services.

This ensures that traffic remains entirely within the AWS private network without traversing the public Internet.

#### Benefits

- Eliminates unnecessary Internet traffic.
- Reduces the risk of data exfiltration.
- Prevents unauthorized outbound communications.
- Strengthens overall network security.

#### Monitoring

When combined with **VPC Flow Logs**, organizations can detect suspicious activities such as:

- Port Scanning
- Network Scanning
- SSH Brute Force
- Connections to unknown IP addresses

### 3. Integrating Shift Left Security into CI/CD Pipelines

#### Problem

When security testing is performed only after deployment, remediation costs increase and vulnerable applications are more likely to reach production environments.

#### Analysis

Shift Left Security integrates security validation into the earliest stages of the software development lifecycle.

Common techniques include:

- Static Application Security Testing (SAST)
- Secret Scanning
- Dependency Scanning
- Infrastructure as Code (IaC) Scanning
- Container Image Scanning

#### Benefits

This approach enables early detection of:

- Hardcoded Credentials
- SQL Injection
- Cross-Site Scripting (XSS)
- Vulnerable Libraries
- Infrastructure Misconfigurations

As a result, security risks are significantly reduced before deployment.

### 4. Centralized Security Monitoring with AWS Security Reference Architecture (SRA)

#### Problem

Organizations operating multiple AWS accounts often face fragmented security monitoring, making incident detection and investigation more difficult.

#### Analysis

AWS Security Reference Architecture (SRA) recommends establishing a dedicated Security Account to centralize security monitoring across the AWS environment.

Common integrated services include:

- Amazon GuardDuty
- AWS Security Hub
- Amazon Macie
- AWS CloudTrail
- AWS Config

#### Figure 2. AWS Security Reference Architecture (SRA)

![AWS Security Reference Architecture](/images/h2.jpg)

**Figure 2** illustrates the AWS Security Reference Architecture (SRA). The architecture is organized into multiple Organizational Units (OUs), with each OU responsible for a specific aspect of governance and security.

The primary components include:

- **Organization Management**: Manages AWS accounts, CloudTrail, IAM Identity Center, and AWS Config.
- **OU – Security**: Centralizes security services such as Amazon GuardDuty, AWS Security Hub, Amazon Macie, AWS Config, and AWS Lambda for automated incident response.
- **OU – Infrastructure**: Deploys infrastructure security services including AWS WAF, AWS Shield Advanced, AWS Network Firewall, Amazon Route 53, and Amazon CloudFront to protect networking and application layers.
- **OU – Workloads**: Hosts application resources including Amazon EC2, Amazon Aurora, Amazon S3, AWS Secrets Manager, AWS KMS, and Amazon Cognito.

This architecture standardizes security management across multiple AWS accounts while consolidating security telemetry into a centralized Security Account, enabling faster detection and response to security incidents.

#### Benefits

This architecture enables:

- Centralized security management.
- Organization-wide visibility.
- Security event correlation.
- Reduced Mean Time To Detect (MTTD).
- Reduced Mean Time To Respond (MTTR).

#### Summary of Security Principles

| Security Principle | AWS Service / Concept | Objective |
|--------------------|-----------------------|-----------|
| Temporary Elevated Access | IAM Role | Prevent privilege escalation |
| Private Connectivity | Amazon VPC Endpoints | Eliminate unnecessary Internet access |
| Shift Left Security | CI/CD Security Pipeline | Detect vulnerabilities early |
| Centralized Monitoring | AWS Security Reference Architecture (SRA) | Improve threat detection and incident response |

### Conclusion

Security on AWS is not solely determined by deploying security services; it is fundamentally influenced by architectural design decisions made from the beginning.

Principles such as temporary privileged access, network isolation, integrating security testing into the software development lifecycle, and centralized security monitoring collectively reduce the attack surface, limit lateral movement, and improve incident detection and response capabilities.

Applying these principles through a **Security by Design** approach enables organizations to build more resilient AWS environments capable of resisting ransomware attacks and other modern cybersecurity threats.


### References

1. https://aws.amazon.com/.../lets-architect.../...
2. https://aws.amazon.com/.../lets-architect.../...

### Article Link

1. https://www.facebook.com/groups/awsstudygroupfcj/permalink/2195119847919642/?rdid=0ofV8lHxkOypQfWq#