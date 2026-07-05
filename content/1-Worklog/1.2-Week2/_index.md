---
title: "Week 2 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.2. </b> "
---


### Week 2 Objectives:

* Understand billing to avoid unexpected costs during practice.
* Learn and understand User, Role, Policy.
* Get familiar with hands-on practice.

### Tasks to be carried out this week:
| Day | Tasks                                                                                                                                                                                                                                                                         | Start Date | End Date   | Resources                                 |
| --- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- | ---------- | ------------------------------------------ |
| 2   | - Basic networking knowledge with Amazon Virtual Private Cloud (VPC): <br>&emsp; + Design network architecture with Subnets (Public/Private) and IP Addressing. <br>&emsp; + Configure Route Tables, Internet Gateway to manage traffic flow. <br>&emsp; + Understand and differentiate Network ACLs and Security Groups. | 27/04/2026 | 27/04/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3   | - Basic server knowledge (EC2): <br>&emsp; + Launch Instance, choose appropriate CPU/RAM configuration based on requirements <br>&emsp; + Manage security using Key Pair and Inbound/Outbound rules.                                                                                                            | 28/04/2026 | 28/04/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4   | - Basic server knowledge (EC2): <br>&emsp; + Launch Instance, choose appropriate CPU/RAM configuration based on requirements <br>&emsp; + Manage security using Key Pair and Inbound/Outbound rules.                                                                                                            | 29/04/2026 | 29/04/2026 | <https://cloudjourney.awsstudygroup.com/> |> |
| 5   | - Authorization with IAM Roles: <br>&emsp; + Create Roles for EC2 to securely execute tasks. <br>&emsp; + Attach Policies to access internal services such as S3 or CloudWatch.                                                                                                                                | 30/04/2026 | 30/04/2026 | <https://cloudjourney.awsstudygroup.com/> |> |
| 6   | - Monitoring with CloudWatch: <br>&emsp; + Monitor metrics such as CPU Utilization, Network In/Out. <br>&emsp; + View system activity logs. <br>&emsp; + Set up Alarms to automatically send email alerts when issues occur.                                                                                 | 31/04/2026 | 31/04/2026 | <https://cloudjourney.awsstudygroup.com/> |> |

### Week 2 Achievements:

* Basic networking knowledge with Amazon VPC:
  * Designed custom network architecture with appropriate CIDR IP ranges.
  * Differentiated and configured Public Subnet for Web Server and Private Subnet for Database.
  * Configured Route Tables, Internet Gateway, and NAT Gateway to manage traffic flow.
  * Used Network ACL and Security Groups to implement multiple layers of security.

* Basic server knowledge with Amazon EC2:
  * Selected Instance Families (T, M, C, R) based on CPU or RAM optimization needs.
  * Practiced launching, managing instance lifecycle, and using User Data for automated setup during boot.
  * Managed storage with EBS (Elastic Block Store) and understood volume types (gp2, gp3, io1).

* Execution authorization with IAM Roles for EC2:
  * Created IAM Roles and attached security Policies following the principle of least privilege.
  * Assigned Roles to EC2 for secure access to S3, DynamoDB without storing Access Keys on the server.
  * Tested and debugged permissions using IAM Policy Simulator.

* System monitoring with Amazon CloudWatch:
  * Monitored basic metrics (CPU, Disk, Network) and installed Unified CloudWatch Agent to collect RAM metrics.
  * Set up CloudWatch Alarms to automatically send notifications via SNS when thresholds are exceeded.
  * Analyzed system and application logs using CloudWatch Logs Insights.