---
title : "Clean Up Resources"
date : "2026-07-07"
weight : 13
chapter : false
pre : " <b> 5.13 </b> "
---

After finishing the workshop, delete resources in this order to avoid ongoing charges:

#### High priority (recurring charges)

1. **AWS WAF on Amplify** (~$23/month)
   - Amplify → app → **Hosting → Firewall** → **Actions** → **Remove firewall**

2. **Route 53 Hosted Zone** ($0.50/month)
   - Route 53 → **Hosted zones** → select the zone → delete the records you created (except NS and SOA) → **Delete zone**

3. **EventBridge Scheduler** (stop the cron first)
   - EventBridge → **Scheduler → Schedules** → select `daily-followup-check` → **Delete**

#### Remaining resources

4. **Amplify app**
   - Amplify → app → **App settings → General** → **Delete app**

5. **API Gateway**
   - API Gateway → select `job-tracker-api` → **Delete**

6. **Lambda functions** (4 functions)
   - Lambda → select each function → **Actions → Delete**
   - IAM → **Roles** → delete the `job-tracker-*-role-*` and `lambda-followup-checker-role-*` roles

7. **DynamoDB**
   - DynamoDB → `JobTrackerJobs` table → turn off **Deletion protection** first → **Delete table**

8. **S3 buckets** (must be empty before deletion)
   - CV bucket: **Empty** → **Delete**
   - CloudTrail logs bucket: **Empty** → **Delete**

9. **CloudTrail**
   - CloudTrail → **Trails** → select `job-tracker-trail` → **Delete**

10. **CloudWatch**
    - **Alarms** → delete `api-5xx-alarm`
    - **Log groups** → delete the `/aws/lambda/...` log groups

11. **SNS / SQS**
    - SNS → delete the `job-tracker-alerts` topic
    - SQS → delete the `followup-dlq` queue

12. **Cognito**
    - Cognito → select the user pool → **Delete user pool**

13. **SES**
    - SES → **Identities** → delete the verified email addresses

#### Worth keeping

- **The budget alert** — free and useful for future projects
- **The GitHub repository** — your code and working history

{{% notice tip %}}
After a day or two, check **Billing → Bills** again: every cost line at $0 means the cleanup is complete. If something still shows up, use **Cost Explorer** to find the leftover service.
{{% /notice %}}
