---
title : "Configure Amazon Cognito"
date : "2026-07-07"
weight : 3
chapter : false
pre : " <b> 5.3 </b> "
---

#### Content
- [Step 1: Create a User Pool](#step-1-create-a-user-pool)
- [Step 2: Record the key values](#step-2-record-the-key-values)
- [Step 3: Enable the auth flow for testing](#step-3-enable-the-auth-flow-for-testing)
- [Step 4: Create a test user](#step-4-create-a-test-user)

{{% notice info %}}
Make sure the selected region is **ap-southeast-1 (Singapore)** — check the top-right corner of the Console before each step.
{{% /notice %}}

#### Step 1: Create a User Pool

1. Go to **Amazon Cognito** → **Create user pool**
2. **Application type**: choose **Single-page application (SPA)**
3. **Name your application**: `job-tracker-web`
4. **Options for sign-in identifiers**: check **Email**
5. **Required attributes for sign-up**: select **email**
6. Choose **Create user directory**

![Cognito User Pool](/images/5-Workshop/5.3-Cognito/cognito-pool.png)

#### Step 2: Record the key values

Once created, write these three values into `NOTES.md` — you will use them throughout the project:

| Value | Where to find it |
|---|---|
| **User Pool ID** | User pool overview page (format `ap-southeast-1_xxxxxxx`) |
| **Client ID** | **App clients** tab → select the client just created |
| **Region** | `ap-southeast-1` |

#### Step 3: Enable the auth flow for testing

To obtain a JWT token from Postman without a frontend:

1. Open the user pool → **App clients** tab → select the client → **Edit**
2. Under **Authentication flows**, also check **`ALLOW_USER_PASSWORD_AUTH`**
3. Choose **Save changes**

#### Step 4: Create a test user

1. Go to the **Users** tab → **Create user**
2. Enter your email and set a password
3. Check **Mark email address as verified**
4. Choose **Create user**

If the user is stuck in `FORCE_CHANGE_PASSWORD`, run this in **CloudShell** to finalize the password:

```bash
aws cognito-idp admin-set-user-password \
  --user-pool-id <USER_POOL_ID> \
  --username <email> \
  --password '<password>' \
  --permanent \
  --region ap-southeast-1
```

{{% notice tip %}}
**IdToken vs AccessToken** — When you later configure the JWT authorizer with `Audience = Client ID`, only the **IdToken** carries a matching `aud` claim. Using the AccessToken returns a 401 even though the token is perfectly valid. This is a very common mistake when testing with Postman.
{{% /notice %}}
