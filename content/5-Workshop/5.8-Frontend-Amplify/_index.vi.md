---
title : "Frontend React và deploy bằng Amplify"
date : "2026-07-07"
weight : 8
chapter : false
pre : " <b> 5.8 </b> "
---

#### Nội dung
- [Bước 1: Khởi tạo project](#bước-1-khởi-tạo-project)
- [Bước 2: Cấu trúc thư mục](#bước-2-cấu-trúc-thư-mục)
- [Bước 3: Cấu hình biến môi trường](#bước-3-cấu-hình-biến-môi-trường)
- [Bước 4: Module xác thực](#bước-4-module-xác-thực)
- [Bước 5: Module gọi API](#bước-5-module-gọi-api)
- [Bước 6: Deploy lên AWS Amplify](#bước-6-deploy-lên-aws-amplify)
- [Bước 7: Cập nhật CORS](#bước-7-cập-nhật-cors)

#### Bước 1: Khởi tạo project

```bash
npm create vite@latest job-tracker-web -- --template react-ts
cd job-tracker-web
npm install
npm install tailwindcss @tailwindcss/vite
```

Sửa `vite.config.ts`:

```typescript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  plugins: [react(), tailwindcss()],
});
```

Thay toàn bộ nội dung `src/index.css` bằng một dòng:

```css
@import "tailwindcss";
```

Chạy `npm run dev` → mở `http://localhost:5173` để kiểm tra.

#### Bước 2: Cấu trúc thư mục

```
src/
├── lib/
│   ├── config.ts     # đọc biến môi trường
│   ├── auth.ts       # gọi Cognito, quản lý token
│   └── api.ts        # gọi 6 route backend
├── pages/
│   ├── AuthPage.tsx  # đăng nhập / đăng ký / xác nhận
│   └── JobsPage.tsx  # danh sách job, CRUD, upload CV
├── components/
│   └── JobForm.tsx   # form tạo/sửa job
├── types.ts          # kiểu dữ liệu dùng chung
├── App.tsx
└── main.tsx
```

#### Bước 3: Cấu hình biến môi trường

Tạo file `.env` ở gốc project (**không commit**):

```env
VITE_API_URL=https://xxxxx.execute-api.ap-southeast-1.amazonaws.com
VITE_COGNITO_REGION=ap-southeast-1
VITE_USER_POOL_CLIENT_ID=<CLIENT_ID>
```

Thêm `.env` vào `.gitignore`, và tạo `.env.example` (có commit) để đồng đội biết cần điền gì.

Tạo `src/lib/config.ts`:

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
Vite nhúng biến `VITE_*` **thẳng vào bundle JS** khi build — người dùng cuối đọc được. Điều này chấp nhận được vì API URL và Client ID vốn là thông tin công khai (app client SPA không có secret). **Tuyệt đối không đặt AWS Access Key vào frontend** — mọi quyền hạn đi qua Cognito token.
{{% /notice %}}

#### Bước 4: Module xác thực

Tạo `src/lib/auth.ts` — gọi thẳng Cognito REST API, tự refresh token:

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
  if (!res.ok) throw new Error(data.message || data.__type || "Lỗi Cognito");
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
    idToken: r.IdToken,          // dùng IdToken, KHÔNG phải AccessToken
    refreshToken: r.RefreshToken,
    expiresAt: Date.now() + r.ExpiresIn * 1000,
    email,
  });
}

// Tự làm mới token khi còn dưới 2 phút
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

#### Bước 5: Module gọi API

Tạo `src/lib/api.ts`:

```typescript
import { CONFIG } from "./config";
import { getIdToken, signOut } from "./auth";
import type { Job, JobStatus } from "../types";

async function call<T>(method: string, path: string, body?: object): Promise<T> {
  const token = await getIdToken();
  if (!token) {
    signOut();
    window.location.reload();
    throw new Error("Phiên đăng nhập đã hết hạn");
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
  if (!res.ok) throw new Error((data as any).message || `Lỗi ${res.status}`);
  return data as T;
}

export const listJobs = (status?: JobStatus) =>
  call<{ count: number; items: Job[] }>("GET", status ? `/jobs?status=${status}` : "/jobs");

export const createJob = (job: Partial<Job>) => call<Job>("POST", "/jobs", job);
export const updateJob = (jobId: string, patch: Partial<Job>) =>
  call<Job>("PUT", `/jobs/${jobId}`, patch);
export const deleteJob = (jobId: string) =>
  call<{ message: string }>("DELETE", `/jobs/${jobId}`);

// Luồng CV: xin URL rồi upload thẳng lên S3
export async function uploadCV(jobId: string, file: File): Promise<void> {
  const { uploadUrl } = await call<{ uploadUrl: string }>(
    "POST", `/jobs/${jobId}/cv-url`, { mode: "upload", fileName: file.name }
  );
  const res = await fetch(uploadUrl, {
    method: "PUT",
    headers: { "Content-Type": "application/pdf" },  // KHÔNG kèm Authorization
    body: file,
  });
  if (!res.ok) throw new Error("Upload CV lên S3 thất bại");
}

export async function downloadCV(jobId: string): Promise<void> {
  const { downloadUrl } = await call<{ downloadUrl: string }>(
    "POST", `/jobs/${jobId}/cv-url`, { mode: "download" }
  );
  window.open(downloadUrl, "_blank");
}
```

Sau đó xây dựng `AuthPage.tsx` (đăng nhập/đăng ký/xác nhận mã) và `JobsPage.tsx` (danh sách job, lọc trạng thái, form CRUD, upload/tải CV) — code đầy đủ trong repo dự án.

![Màn đăng nhập](/images/5-Workshop/5.8-Frontend-Amplify/ui-login.png)

![Màn quản lý job](/images/5-Workshop/5.8-Frontend-Amplify/ui-jobs.png)

#### Bước 6: Deploy lên AWS Amplify

{{% notice info %}}
**AWS Amplify Hosting** đóng gói sẵn hosting tĩnh + CDN toàn cầu + HTTPS + CI/CD. So với tự dựng S3 + CloudFront, Amplify gọn hơn nhiều cho một frontend tĩnh.
{{% /notice %}}

1. Push code lên GitHub (branch `main`)
2. Vào **AWS Amplify** → **Create new app**
3. **Source code provider**: chọn **GitHub** → **Authorize** (chỉ cấp quyền cho repo cần thiết)
4. Chọn **repository** và **branch** `main` → **Next**
5. **App settings**:
   - **Frontend build command**: `npm run build`
   - **Build output directory**: `dist` (Vite dùng `dist`, không phải `build`)
6. Mở **Advanced settings** → **Environment variables** → thêm 3 biến:

| Key | Value |
|---|---|
| `VITE_API_URL` | Invoke URL của API Gateway |
| `VITE_COGNITO_REGION` | `ap-southeast-1` |
| `VITE_USER_POOL_CLIENT_ID` | Client ID của Cognito |

7. **Next** → **Save and deploy**

{{% notice warning %}}
File `.env` **không được commit** lên GitHub, nên Amplify sẽ không có biến môi trường nếu bạn không khai ở bước 6. Thiếu bước này, app build xong nhưng đăng nhập và gọi API đều lỗi.
{{% /notice %}}

Pipeline chạy 4 giai đoạn (Provision → Build → Deploy → Verify), khoảng 3–5 phút. Sau đó Amplify cấp URL dạng `https://main.xxxxx.amplifyapp.com`.

![Amplify deploy](/images/5-Workshop/5.8-Frontend-Amplify/amplify-deploy.png)

#### Bước 7: Cập nhật CORS

Thêm domain Amplify vào danh sách origin được phép:

1. **API Gateway** → `job-tracker-api` → **CORS** → thêm `https://main.xxxxx.amplifyapp.com` vào **Access-Control-Allow-Origin** → **Save**
2. **S3 bucket CV** → **Permissions** → **CORS** → thêm domain đó vào `AllowedOrigins` → **Save**

**Kiểm thử cuối:** mở URL Amplify → đăng nhập → tạo/sửa/xoá job → upload và tải CV → nhấn F5 giữa trang (không bị lỗi 404 nhờ Amplify tự xử lý SPA routing).
