# 📘 Blink Technical Documentation

> Living document of Blink's system architecture and implementation details.  
> Last updated: 2025-06-01

---

## 🧩 Modules Overview

This documentation breaks down the key components of Blink's technical infrastructure. Each module is designed to reflect Blink’s core values: safety, respect, and meaningful connection.

### Covered Modules

- [x] Authentication
- [ ] User Profile
- [ ] Matching Engine
- [ ] Conversation Flow
- [ ] AI Onboarding
- [ ] Moderation & Reporting
- [ ] Notifications
- [ ] Admin & Metrics
- [ ] Infrastructure & Deployment

---

## 🔐 Authentication Module

Blink uses [Supabase](https://supabase.com/) as the core authentication provider, integrated with a custom backend API to offer enhanced control, flexibility, and extensibility.

### Key Principles

- Minimal friction for users  
- Secure token management  
- Unified backend logic for critical flows  
- Compliance with Blink's safety-first philosophy

---

### 🔄 Login Flow

- **Method**: Supabase client SDK (`@supabase/supabase-js`)
- **Token Handling**: Supabase handles access and refresh tokens securely.
- **Session Persistence**: Managed client-side using Supabase's session observer.

**Steps:**

1. User inputs email and password in the app.  
2. App uses `supabase.auth.signInWithPassword()`.  
3. Supabase returns `access_token` and `refresh_token`.  
4. Tokens are securely stored on the client.  
5. Session state is synced with `onAuthStateChange()`.  

---

### 🆕 Registration Flow

- **Method**: Custom Backend API
- **Endpoint**: `POST /auth/register`
- **Backend Role**: Uses Supabase Admin SDK to create the user.

**Example Request:**

POST /auth/register
Content-Type: application/json

{
"email": "user@example.com",
"password": "secure_password",
"name": "User Name"
}


**Backend Actions:**

- Validates email and password  
- Creates the user in Supabase  
- Saves custom metadata (e.g., name, preferences)  
- Sends confirmation email if enabled  

---

### 🔢 OTP Verification

- **Flow**: Email-based OTP via backend  
- **Supabase Feature**: Magic Link or OTP (email)  
- **Frontend**: Collects and sends the OTP to the backend  
- **Backend**: Verifies via Supabase Admin API  

**Benefits:**

- Allows login without password  
- Can be used for verification or 2FA  
- Enables branded and custom OTP flows  

---

### 🔁 Password Management

#### Forgot Password

- **Endpoint**: `POST /auth/forgot-password`
- **Action**: Sends reset email using Supabase

**Request Example:**

POST /auth/forgot-password
Content-Type: application/json

{
"email": "user@example.com"
}


#### Change Password

- **Endpoint**: `POST /auth/change-password`
- **Requires Auth**: Yes (must be logged in)

**Request Example:**

POST /auth/change-password
Content-Type: application/json

{
"currentPassword": "old_pass",
"newPassword": "new_pass"
}


---

### 🔐 Security Considerations

- ✅ Email confirmation required  
- ✅ Row-Level Security (RLS) enabled in Supabase  
- ✅ Rate limiting on all sensitive endpoints  
- ✅ Full input validation server-side  
- ✅ Optional 2FA/MFA in roadmap  
- ✅ Tokens stored in HttpOnly cookies or encrypted storage  

---

### 🧪 Tests & Validations

| Flow              | Entry Point       | Status |
|------------------|-------------------|--------|
| Login             | Client SDK        | ✅     |
| Register          | Backend API       | ✅     |
| OTP Login         | Backend API       | ✅     |
| Forgot Password   | Backend API       | ✅     |
| Change Password   | Backend API       | ✅     |

---

### 📌 To Do / Improvements

- [ ] Add social login (Google, Apple)  
- [ ] Integrate optional MFA (TOTP)  
- [ ] Add brute-force protection and lockouts  
- [ ] Improve session renewal UX on mobile  
- [ ] Admin override: password reset / disable user  

---

## 🛠 Notes for the Development Team

- Maintain parity between frontend and backend validations  
- Implement monitoring on all auth endpoints (metrics + alerts)  
- Keep Supabase and SDKs up to date for security patches  
- Never expose Supabase Admin API on the frontend  

---
