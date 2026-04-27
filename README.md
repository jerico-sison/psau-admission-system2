# PSAU Admission System

AI-assisted admission management system for Pampanga State Agricultural University (PSAU).

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

## Local Setup

1. Place project in your web root (for example, `xampp/htdocs`).
2. Configure DB environment variables (or adjust fallback values carefully).
3. Import your database schema (project docs reference `database/psau_admission.sql`).
4. Ensure runtime-writable directories exist and are writable (for example, `uploads`, `images`, `logs` where applicable).
5. Open `public/` entry pages from your local server.

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