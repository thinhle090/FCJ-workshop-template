---
title : "React Frontend and Amplify Deployment"
date : "2026-07-07"
weight : 8
chapter : false
pre : " <b> 5.8 </b> "
---

#### Content
- [Step 1: Initialize the project](#step-1-initialize-the-project)
- [Step 2: Folder structure](#step-2-folder-structure)
- [Step 3: Environment configuration](#step-3-environment-configuration)
- [Step 4: The auth module](#step-4-the-auth-module)
- [Step 5: The API module](#step-5-the-api-module)
- [Step 6: Deploy to AWS Amplify](#step-6-deploy-to-aws-amplify)
- [Step 7: Update CORS](#step-7-update-cors)

#### Step 1: Initialize the project

```bash
npm create vite@latest job-tracker-web -- --template react-ts
cd job-tracker-web
npm install
npm install tailwindcss @tailwindcss/vite
```

Edit `vite.config.ts`:

```typescript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  plugins: [react(), tailwindcss()],
});
```

Replace the entire contents of `src/index.css` with one line:

```css
@import "tailwindcss";
```

Run `npm run dev` and open `http://localhost:5173` to verify.

#### Step 2: Folder structure

```
src/
├── lib/
│   ├── config.ts     # reads environment variables
│   ├── auth.ts       # Cognito calls, token management
│   └── api.ts        # calls the 6 backend routes
├── pages/
│   ├── AuthPage.tsx  # sign in / sign up / confirm
│   └── JobsPage.tsx  # job list, CRUD, CV upload
├── components/
│   └── JobForm.tsx   # create/edit job form
├── types.ts          # shared type definitions
├── App.tsx
└── main.tsx
```

#### Step 3: Environment configuration

Create a `.env` file at the project root (**do not commit it**):

```env
VITE_API_URL=https://xxxxx.execute-api.ap-southeast-1.amazonaws.com
VITE_COGNITO_REGION=ap-southeast-1
VITE_USER_POOL_CLIENT_ID=<CLIENT_ID>
```

Add `.env` to `.gitignore`, and create a committed `.env.example` so teammates know what to fill in.

Create `src/lib/config.ts`:

```typescript
export const CONFIG = {
  API_URL: import.meta.env.VITE_API_URL as string,
  COGNITO_REGION: import.meta.env.VITE_COGNITO_REGION as string,
  USER_POOL_CLIENT_ID: import.meta.env.VITE_USER_POOL_CLIENT_ID as string,
};

export const COGNITO_ENDPOINT =
  `https://cognito-idp.${CONFIG.COGNITO_REGION}.amazonaws.com/`;
```

{{% notice warning %}}
Vite bakes `VITE_*` variables **directly into the JS bundle** at build time — end users can read them. That is acceptable here because the API URL and Client ID are public by design (an SPA app client has no secret). **Never put AWS Access Keys in the frontend** — all authorization flows through the Cognito token.
{{% /notice %}}

#### Step 4: The auth module

Create `src/lib/auth.ts` — it calls the Cognito REST API directly and refreshes tokens automatically:

```typescript
import { CONFIG, COGNITO_ENDPOINT } from "./config";

const KEY = "jobtracker_auth";

interface AuthState {
  idToken: string;
  refreshToken: string;
  expiresAt: number;
  email: string;
}

async function cognito(target: string, body: object) {
  const res = await fetch(COGNITO_ENDPOINT, {
    method: "POST",
    headers: {
      "Content-Type": "application/x-amz-json-1.1",
      "X-Amz-Target": `AWSCognitoIdentityProviderService.${target}`,
    },
    body: JSON.stringify(body),
  });
  const data = await res.json();
  if (!res.ok) throw new Error(data.message || data.__type || "Cognito error");
  return data;
}

const save = (a: AuthState) => localStorage.setItem(KEY, JSON.stringify(a));
const load = (): AuthState | null => {
  try { return JSON.parse(localStorage.getItem(KEY) ?? "null"); }
  catch { return null; }
};

export function signUp(email: string, password: string) {
  return cognito("SignUp", {
    ClientId: CONFIG.USER_POOL_CLIENT_ID,
    Username: email,
    Password: password,
    UserAttributes: [{ Name: "email", Value: email }],
  });
}

export function confirmSignUp(email: string, code: string) {
  return cognito("ConfirmSignUp", {
    ClientId: CONFIG.USER_POOL_CLIENT_ID,
    Username: email,
    ConfirmationCode: code,
  });
}

export async function signIn(email: string, password: string) {
  const data = await cognito("InitiateAuth", {
    AuthFlow: "USER_PASSWORD_AUTH",
    ClientId: CONFIG.USER_POOL_CLIENT_ID,
    AuthParameters: { USERNAME: email, PASSWORD: password },
  });
  const r = data.AuthenticationResult;
  save({
    idToken: r.IdToken,          // use the IdToken, NOT the AccessToken
    refreshToken: r.RefreshToken,
    expiresAt: Date.now() + r.ExpiresIn * 1000,
    email,
  });
}

// Refresh the token when less than 2 minutes remain
async function refresh(a: AuthState): Promise<AuthState> {
  const data = await cognito("InitiateAuth", {
    AuthFlow: "REFRESH_TOKEN_AUTH",
    ClientId: CONFIG.USER_POOL_CLIENT_ID,
    AuthParameters: { REFRESH_TOKEN: a.refreshToken },
  });
  const r = data.AuthenticationResult;
  const next = { ...a, idToken: r.IdToken, expiresAt: Date.now() + r.ExpiresIn * 1000 };
  save(next);
  return next;
}

export async function getIdToken(): Promise<string | null> {
  let a = load();
  if (!a) return null;
  if (Date.now() > a.expiresAt - 2 * 60 * 1000) {
    try { a = await refresh(a); }
    catch { signOut(); return null; }
  }
  return a.idToken;
}

export const currentUser = () => load()?.email ?? null;
export const signOut = () => localStorage.removeItem(KEY);
```

#### Step 5: The API module

Create `src/lib/api.ts`:

```typescript
import { CONFIG } from "./config";
import { getIdToken, signOut } from "./auth";
import type { Job, JobStatus } from "../types";

async function call<T>(method: string, path: string, body?: object): Promise<T> {
  const token = await getIdToken();
  if (!token) {
    signOut();
    window.location.reload();
    throw new Error("Session expired");
  }
  const res = await fetch(`${CONFIG.API_URL}${path}`, {
    method,
    headers: {
      Authorization: `Bearer ${token}`,
      ...(body ? { "Content-Type": "application/json" } : {}),
    },
    body: body ? JSON.stringify(body) : undefined,
  });
  const data = await res.json().catch(() => ({}));
  if (!res.ok) throw new Error((data as any).message || `Error ${res.status}`);
  return data as T;
}

export const listJobs = (status?: JobStatus) =>
  call<{ count: number; items: Job[] }>("GET", status ? `/jobs?status=${status}` : "/jobs");

export const createJob = (job: Partial<Job>) => call<Job>("POST", "/jobs", job);
export const updateJob = (jobId: string, patch: Partial<Job>) =>
  call<Job>("PUT", `/jobs/${jobId}`, patch);
export const deleteJob = (jobId: string) =>
  call<{ message: string }>("DELETE", `/jobs/${jobId}`);

// CV flow: request a URL, then upload straight to S3
export async function uploadCV(jobId: string, file: File): Promise<void> {
  const { uploadUrl } = await call<{ uploadUrl: string }>(
    "POST", `/jobs/${jobId}/cv-url`, { mode: "upload", fileName: file.name }
  );
  const res = await fetch(uploadUrl, {
    method: "PUT",
    headers: { "Content-Type": "application/pdf" },  // NO Authorization header
    body: file,
  });
  if (!res.ok) throw new Error("Failed to upload CV to S3");
}

export async function downloadCV(jobId: string): Promise<void> {
  const { downloadUrl } = await call<{ downloadUrl: string }>(
    "POST", `/jobs/${jobId}/cv-url`, { mode: "download" }
  );
  window.open(downloadUrl, "_blank");
}
```

Then build `AuthPage.tsx` (sign in/sign up/confirmation code) and `JobsPage.tsx` (job list, status filter, CRUD form, CV upload/download) — full code is in the project repository.

![Login screen](/images/5-Workshop/5.8-Frontend-Amplify/ui-login.png)

![Jobs screen](/images/5-Workshop/5.8-Frontend-Amplify/ui-jobs.png)

#### Step 6: Deploy to AWS Amplify

{{% notice info %}}
**AWS Amplify Hosting** bundles static hosting + a global CDN + HTTPS + CI/CD. Compared to wiring up S3 + CloudFront manually, Amplify is far simpler for a static frontend.
{{% /notice %}}

1. Push the code to GitHub (branch `main`)
2. Go to **AWS Amplify** → **Create new app**
3. **Source code provider**: choose **GitHub** → **Authorize** (grant access only to the needed repo)
4. Select the **repository** and branch `main` → **Next**
5. **App settings**:
   - **Frontend build command**: `npm run build`
   - **Build output directory**: `dist` (Vite outputs `dist`, not `build`)
6. Open **Advanced settings** → **Environment variables** → add three variables:

| Key | Value |
|---|---|
| `VITE_API_URL` | Your API Gateway Invoke URL |
| `VITE_COGNITO_REGION` | `ap-southeast-1` |
| `VITE_USER_POOL_CLIENT_ID` | Your Cognito Client ID |

7. **Next** → **Save and deploy**

{{% notice warning %}}
The `.env` file is **not committed** to GitHub, so Amplify has no environment variables unless you declare them in step 6. Skip this and the app builds fine but sign-in and API calls will all fail.
{{% /notice %}}

The pipeline runs four phases (Provision → Build → Deploy → Verify), roughly 3–5 minutes. Amplify then issues a URL like `https://main.xxxxx.amplifyapp.com`.

![Amplify deploy](/images/5-Workshop/5.8-Frontend-Amplify/amplify-deploy.png)

#### Step 7: Update CORS

Add the Amplify domain to the allowed origins:

1. **API Gateway** → `job-tracker-api` → **CORS** → add `https://main.xxxxx.amplifyapp.com` to **Access-Control-Allow-Origin** → **Save**
2. **S3 CV bucket** → **Permissions** → **CORS** → add the same domain to `AllowedOrigins` → **Save**

**Final test:** open the Amplify URL → sign in → create/edit/delete a job → upload and download a CV → press F5 mid-page (no 404, because Amplify handles SPA routing automatically).
