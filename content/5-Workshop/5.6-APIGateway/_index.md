---
title : "Configure API Gateway"
date : "2026-07-07"
weight : 6
chapter : false
pre : " <b> 5.6 </b> "
---

#### Content
- [Step 1: Create the HTTP API](#step-1-create-the-http-api)
- [Step 2: Create the routes](#step-2-create-the-routes)
- [Step 3: Create the JWT Authorizer](#step-3-create-the-jwt-authorizer)
- [Step 4: Attach integrations](#step-4-attach-integrations)
- [Step 5: Configure CORS](#step-5-configure-cors)
- [Step 6: Test with Postman](#step-6-test-with-postman)

{{% notice info %}}
This workshop uses an **HTTP API** instead of a REST API: roughly 70% cheaper, one-step CORS configuration, and a native JWT authorizer for Cognito.
{{% /notice %}}

#### Step 1: Create the HTTP API

1. Go to **API Gateway** → **Create API**
2. Under **HTTP API**, choose **Build**
3. **API name**: `job-tracker-api`
4. Skip Integrations → **Next**
5. Skip Routes → **Next** → Skip Stages → **Next** → **Create**

Record the **Invoke URL** (`https://xxxxx.execute-api.ap-southeast-1.amazonaws.com`) in `NOTES.md`.

#### Step 2: Create the routes

In the left menu choose **Routes** → **Create**, then add all six routes:

| Method | Path |
|---|---|
| GET | `/jobs` |
| POST | `/jobs` |
| GET | `/jobs/{jobId}` |
| PUT | `/jobs/{jobId}` |
| DELETE | `/jobs/{jobId}` |
| POST | `/jobs/{jobId}/cv-url` |

{{% notice warning %}}
Type `{jobId}` with curly braces and exact casing — the Lambda reads `event.pathParameters.jobId`; wrong casing yields `undefined`.
{{% /notice %}}

#### Step 3: Create the JWT Authorizer

1. Left menu → **Authorization** → select any route → **Create and attach an authorizer**
2. **Authorizer type**: **JWT**
3. **Name**: `cognito-jwt`
4. **Identity source**: keep the default `$request.header.Authorization`
5. **Issuer URL**: `https://cognito-idp.ap-southeast-1.amazonaws.com/<USER_POOL_ID>`
6. **Audience**: enter the app client's **Client ID**
7. Choose **Create and attach**

Then attach this authorizer to **all six routes**: select each route → **Select existing authorizer** → choose `cognito-jwt` → **Attach authorizer**.

![API Gateway routes](/images/5-Workshop/5.6-APIGateway/apigw-routes.png)

{{% notice warning %}}
Double-check that all six routes show the **JWT Auth** badge. Missing one creates a security gap — requests without a token would reach the Lambda.
{{% /notice %}}

#### Step 4: Attach integrations

Left menu → **Integrations** → select each route → **Create and attach an integration**:

| Route | Lambda function |
|---|---|
| `GET /jobs` | `job-tracker-get-jobs` |
| `GET /jobs/{jobId}` | `job-tracker-get-jobs` |
| `POST /jobs` | `job-tracker-crud-job` |
| `PUT /jobs/{jobId}` | `job-tracker-crud-job` |
| `DELETE /jobs/{jobId}` | `job-tracker-crud-job` |
| `POST /jobs/{jobId}/cv-url` | `job-tracker-cv-presigned` (created in section 5.7) |

#### Step 5: Configure CORS

1. Left menu → **CORS** → **Configure**
2. **Access-Control-Allow-Origin**: enter `http://localhost:5173` → **Add** (the real domain is added after the Amplify deployment)
3. **Access-Control-Allow-Headers**: enter `*` → **Add**
4. **Access-Control-Allow-Methods**: select `*`
5. **Access-Control-Max-Age**: `300`
6. **Access-Control-Allow-Credentials**: leave as **NO**
7. **Save**

{{% notice info %}}
Allow-Credentials stays off because the app sends the JWT in the `Authorization` header, not cookies. Enabling it would also conflict with an `*` origin per the CORS spec.
{{% /notice %}}

#### Step 6: Test with Postman

**Get a JWT token** — create a POST request to `https://cognito-idp.ap-southeast-1.amazonaws.com/`:

- Headers: `Content-Type: application/x-amz-json-1.1`, `X-Amz-Target: AWSCognitoIdentityProviderService.InitiateAuth`
- Body (raw JSON):

```json
{
  "AuthFlow": "USER_PASSWORD_AUTH",
  "ClientId": "<CLIENT_ID>",
  "AuthParameters": {
    "USERNAME": "<email>",
    "PASSWORD": "<password>"
  }
}
```

In **Scripts → Post-response**, add a script to auto-save the token:

```javascript
const res = pm.response.json();
if (res.AuthenticationResult) {
    pm.environment.set("token", res.AuthenticationResult.IdToken);
}
```

**Test the API** — with `Authorization: Bearer {{token}}`:

```bash
# 1. No token → must return 401
GET {{api_url}}/jobs

# 2. Create a job
POST {{api_url}}/jobs
{
  "company": "FPT Software",
  "position": "Cloud Engineer Intern",
  "followUpDate": "2026-07-08"
}

# 3. List jobs
GET {{api_url}}/jobs

# 4. Update
PUT {{api_url}}/jobs/<jobId>
{ "status": "INTERVIEW" }

# 5. Delete
DELETE {{api_url}}/jobs/<jobId>
```

![Postman 401](/images/5-Workshop/5.6-APIGateway/postman-401.png)

**Completion checklist:**
- [ ] Calling the API without a token returns **401 Unauthorized**
- [ ] POST creates a job → **201**, item visible in DynamoDB
- [ ] GET returns the correct list; PUT/DELETE work
- [ ] User A cannot read or modify user B's jobs

{{% notice tip %}}
Make sure you take the **IdToken** from the response (not the `AccessToken`) — only the IdToken has an `aud` claim matching the authorizer's Audience.
{{% /notice %}}
