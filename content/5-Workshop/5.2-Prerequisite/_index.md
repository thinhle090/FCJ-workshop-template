---
title : "Prerequisites"
date : "2026-07-07"
weight : 2
chapter : false
pre : " <b> 5.2 </b> "
---

#### Content
- [Accounts and tools](#accounts-and-tools)
- [Step 1: Set up a Budget Alert](#step-1-set-up-a-budget-alert)
- [Step 2: Prepare the development environment](#step-2-prepare-the-development-environment)
- [Naming conventions](#naming-conventions)

#### Accounts and tools

| Requirement | Notes |
|---|---|
| AWS account | Free Tier with promotional credits |
| IAM user | One per team member, **never use the root user** |
| Node.js 20 LTS | Run `node -v` to verify |
| Git + GitHub account | For version control and Amplify deployment |
| Postman | API testing |
| VS Code | Code editor |

#### Step 1: Set up a Budget Alert

{{% notice warning %}}
Do this **first**, before creating any resource. A budget alert is your insurance against unexpected costs.
{{% /notice %}}

1. Sign in to the AWS Console and open **Billing and Cost Management**
2. In the left menu choose **Budgets** → **Create budget**
3. Select **Zero spend budget** or a **Monthly cost budget** with a **$5** threshold
4. Enter every team member's email to receive alerts
5. Choose **Create budget**

![Budget alert](/images/5-Workshop/5.2-Prerequisite/budget-alert.png)

#### Step 2: Prepare the development environment

1. Install **Node.js 20 LTS** from [nodejs.org](https://nodejs.org), then verify:
```bash
node -v    # v20.x or v22.x
npm -v
```

2. Install **Git** from [git-scm.com](https://git-scm.com), verify with `git -v`

3. Create a GitHub repository (keep it **Private** during development) and invite members via **Settings → Collaborators**

4. Agree on a team workflow: **one branch per module + Pull Request**
```bash
git checkout main && git pull
git checkout -b feat/module-name
# ... code, commit ...
git push -u origin feat/module-name
# Open a Pull Request on GitHub, have a teammate review, then merge
```

#### Naming conventions

Agree on these upfront to avoid confusion when configuring IAM and environment variables:

| Resource | Name |
|---|---|
| DynamoDB table | `JobTrackerJobs` |
| DynamoDB GSI | `FollowUpIndex` |
| Lambda | `job-tracker-crud-job`, `job-tracker-get-jobs`, `job-tracker-cv-presigned`, `lambda-followup-checker` |
| API Gateway | `job-tracker-api` |
| S3 CV bucket | `job-tracker-cv-<team-name>-2026` |
| SQS queue | `followup-dlq` |
| SNS topic | `job-tracker-alerts` |
| CloudTrail trail | `job-tracker-trail` |
| Amplify app | `job-tracker-web` |

{{% notice tip %}}
Create a `NOTES.md` file in your repo to record key values: User Pool ID, Client ID, API Invoke URL, bucket names, Account ID. The whole team can refer to it.
{{% /notice %}}
