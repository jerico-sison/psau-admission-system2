# PSAU Admission System

AI-assisted admission system at Pampanga State Agricultural University (PSAU).

## Overview

This project handles the full admission lifecycle: applicant registration, application submission, verification, exam scheduling, scoring, course assignment, enrollment scheduling, and enrollment completion.

The system is implemented in PHP with server-rendered pages and JavaScript modules, and integrates external services for database, email, and AI features.

## Current Stack (From Code)

- Backend: PHP `>=7.4` (`composer.json`)
- Frontend: HTML, CSS, JavaScript
- Database: MySQL
  - Production default/fallback is Google Cloud SQL host (`34.170.34.174`)
  - Environment-variable driven configuration in `includes/db_connect.php`
- Hosting/deployment config: Render (`render.yaml`, Dockerfiles, Procfile)
- Email delivery: Firebase-hosted email function via `firebase/firebase_email.php`
- Security controls: session auth, password hashing, reCAPTCHA token gating, OTP expiry/rate-limiting helpers
- AI integrations: chatbot and recommendation proxies in `public/ai/*_proxy.php`

## Important Clarifications

- OTP is email-based in current flows (`public/send_email_otp.php`, `admin/send_admin_otp.php`).
- No active SMS sending flow is used for OTP in current implementation.
- Some old references to SMS may still appear in static text, but operational OTP endpoints send email.

## Main Modules

- `public/`: applicant-side pages and endpoints
  - registration/login/forgot password flows
  - application progress and submission features
  - AI chatbot/recommendation pages and proxy endpoints
- `admin/`: admin-side management pages and actions
  - application review and verification
  - exam scheduling and reminders
  - bulk/manual score handling
  - enrollment scheduling/completion
- `includes/`: shared backend utilities
  - database connection and validation
  - auth/session helpers
  - OTP attempt/rate limiting
- `firebase/`: Firebase config and email integration

## Database Configuration

The app reads DB values from environment variables (recommended):

- `DB_HOST`
- `DB_NAME`
- `DB_USER`
- `DB_PASS`
- `DB_PORT`

Fallback values in `includes/db_connect.php` currently point to:

- Host: `34.170.34.174`
- Port: `3306`
- Database: `psau_admission`
- User: `root`

On Render (`RENDER=true`), DB connection options enable MySQL SSL-related settings for production compatibility.

### Example Database Environment Values

Use these as a reference when setting your deployment environment:

```env
DB_HOST=34.170.34.174
DB_PORT=3306
DB_NAME=psau_admission
DB_USER=root
DB_PASS=your_database_password_here
RENDER=true
ENVIRONMENT=production
```

> Keep real database passwords in environment variables only. Do not store real secrets in `README.md` or tracked source files.

## Email / OTP Configuration

Current OTP flow is email-based. The app uses Firebase-related settings plus an email function URL.

### Values Used by the Code

Configured in `firebase/config.php` and/or deployment env vars:

- `FIREBASE_API_KEY`
- `FIREBASE_AUTH_DOMAIN`
- `FIREBASE_PROJECT_ID`
- `FIREBASE_STORAGE_BUCKET`
- `FIREBASE_MESSAGING_SENDER_ID`
- `FIREBASE_APP_ID`
- `FIREBASE_EMAIL_FUNCTION_URL`

### Example Email/OTP Environment Values

```env
FIREBASE_API_KEY=your_firebase_api_key
FIREBASE_AUTH_DOMAIN=your-project.firebaseapp.com
FIREBASE_PROJECT_ID=your-project-id
FIREBASE_STORAGE_BUCKET=your-project.appspot.com
FIREBASE_MESSAGING_SENDER_ID=123456789012
FIREBASE_APP_ID=1:123456789012:web:exampleappid
FIREBASE_EMAIL_FUNCTION_URL=https://your-email-function-url
```

### App-Level User/Admin Credentials

The system does not use a single hardcoded app login email/password in code. Instead:

- applicants and admins authenticate using records in the database
- passwords are stored hashed
- OTP verification is used in registration/reset flows

For first-time admin setup, create an admin account through the admin registration flow and keep credentials private.

### Email Addresses Referenced in Code

These addresses are currently referenced by templates/endpoints:

- `officeofthepresident@psau.edu.ph` (public contact display)
- `system@psau.edu.ph` (system sender/reference in admin flows)
- `noreply@psau-admission-system2.onrender.com` (fallback sender/reply-to in simple email helper)
- `jericogutierrezsison12@gmail.com` (admin OTP restriction and Firebase email sender references)

> If you plan to deploy this to production for a broader team, replace hardcoded contact/sender emails with environment-driven values.

## External Service Endpoints (Current)

Detected in the current codebase:

- Main deployment domain: `https://psau-admission-system2.onrender.com`
- Firebase email function: `https://sendemail-alsstt22ha-uc.a.run.app`
- Chatbot API base: `https://flaskbot-1.onrender.com`
- Recommendation API base: `https://recommender-np4e.onrender.com`
- FAQ data API base: `https://database-dhe2.onrender.com`

## Recommended Secret Handling

- Use environment variables for database passwords, API keys, and service endpoints.
- Never commit production email passwords, database passwords, or private keys.
- Rotate credentials if a secret was ever committed to git history.
- Keep separate credentials for local, staging, and production.

## Local Setup

1. Place project in your web root (for example, `xampp/htdocs`).
2. Configure DB environment variables (or adjust fallback values carefully).
3. Import your database schema (project docs reference `database/psau_admission.sql`).
4. Ensure runtime-writable directories exist and are writable (for example, `uploads`, `images`, `logs` where applicable).
5. Open `public/` entry pages from your local server.

## Full Installation Guide

### 1) Prerequisites

- PHP `7.4` or newer
- MySQL/MariaDB
- Web server (Apache via XAMPP recommended for local)
- Composer (for PHP dependency management)
- Internet access for external services (Firebase + AI endpoints)

### 2) Get the Project

1. Clone/download the project to your server web root.
2. Example local path:
   - `C:\xampp\htdocs\Development of AI-Assisted Admission System at PSAU`

### 3) Install PHP Dependencies

From the project root:

```bash
composer install
```

If Composer is not installed globally, use your local Composer binary.

### 4) Configure Environment Variables

Set these in your deployment platform (Render) or local server env.

#### Required Database Variables

- `DB_HOST`
- `DB_PORT`
- `DB_NAME`
- `DB_USER`
- `DB_PASS`

#### Required App/Runtime Variables

- `ENVIRONMENT` (`production` or `development`)
- `RENDER` (`true` in Render deployments)

#### Required Firebase/Email Variables

- `FIREBASE_API_KEY`
- `FIREBASE_AUTH_DOMAIN`
- `FIREBASE_PROJECT_ID`
- `FIREBASE_STORAGE_BUCKET`
- `FIREBASE_MESSAGING_SENDER_ID`
- `FIREBASE_APP_ID`
- `FIREBASE_EMAIL_FUNCTION_URL`

#### Optional Security Variables

- `RECAPTCHA_SITE_KEY`
- `RECAPTCHA_SECRET_KEY`
- `ALLOWED_HOSTS`

### 5) Import Database

1. Create database (example: `psau_admission`).
2. Import schema SQL referenced by docs (`database/psau_admission.sql`).
3. Verify tables like `users`, `applications`, and OTP-related tables exist.

### 6) Set Folder Permissions

Ensure these are writable by the web process (if used by your flows):

- `uploads/`
- `images/`
- `logs/`

### 7) Start and Verify Locally

1. Start Apache + MySQL.
2. Open app entrypoint:
   - `http://localhost/Development of AI-Assisted Admission System at PSAU/`
3. Test:
   - Login/Registration pages
   - OTP send endpoint behavior
   - Admin pages (with authorized admin account)

## Email Sending Setup (Important)

This project sends OTP and notification emails using Firebase-backed email function integration.

### A) Configure Firebase Project

1. Create/select your Firebase project.
2. Confirm project settings match your configured values (`apiKey`, `authDomain`, etc.).
3. Set/verify the email function endpoint URL in:
   - `FIREBASE_EMAIL_FUNCTION_URL`

### B) Configure Sender Identity

Code currently references sender addresses in multiple places (see Email section above). For production:

1. Decide your official sender email/domain.
2. Update hardcoded sender references to environment-driven values where possible.
3. Ensure sender mailbox/service is authorized by your email function implementation.

### C) Verify Email Delivery

1. Test OTP flows:
   - Applicant OTP (`public/send_email_otp.php`)
   - Admin OTP (`admin/send_admin_otp.php`)
2. Check app logs and Firebase/email function logs for failures.
3. Validate spam/deliverability (SPF/DKIM/DMARC if using custom domain).

## Admin Account Setup

- Admin registration flow is present under `admin/`.
- Current code includes restricted-email checks for admin OTP in specific endpoints.
- If your deployment needs multiple admin emails, adjust restriction logic safely and test OTP flow before production use.

## Deployment Setup (Render)

1. Use `render.yaml` as baseline.
2. Set all required env vars in Render dashboard.
3. Confirm DB network access and SSL compatibility for Google Cloud SQL.
4. Deploy and verify:
   - `/health.php`
   - main domain page
   - login/register flows
   - OTP email sending

## Post-Deployment Checklist

- Database connection works in production.
- Registration + OTP email verification works.
- Forgot password OTP works.
- Admin login and protected routes work.
- AI chatbot/recommendation endpoints respond.
- No secrets are hardcoded in committed files.

## Troubleshooting Quick Notes

- DB errors: recheck `DB_*` values and DB accessibility/firewall.
- OTP not sending: verify `FIREBASE_EMAIL_FUNCTION_URL` and function logs.
- reCAPTCHA issues: verify `RECAPTCHA_*` keys and allowed hosts.
- White screen/500: inspect PHP logs + deployment logs first.

## Deployment Notes

- `render.yaml` includes production environment keys used by this codebase.
- Root has multiple deployment artifacts (`Dockerfile`, `Dockerfile.minimal`, `Dockerfile.simple`, `Procfile`) that support different deployment approaches.
- `health.php` is available for health checks.

## Repository Structure

- `admin/` Admin interface and admission workflow actions
- `public/` Applicant interface and API-like endpoints
- `includes/` Shared PHP business logic/utilities
- `firebase/` Firebase and email integration code
- `images/`, `uploads/`, `logo/` Assets and uploaded files

## License

Proprietary - All rights reserved.