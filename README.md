# CK Reynolds Tax Service Platform

**Full-stack SaaS platform** for a tax preparation business (IRS Enrolled Agent). Production client project — source code is private, this repo showcases the architecture, security posture, and technical decisions.

**Status:** Production-ready. 15 sprints shipped + 8-phase security audit passed.
**Built by:** [D. Michael Piscitelli](https://github.com/herakles-dev) | [herakles.dev](https://herakles.dev)

---

## At a Glance

| Metric | Value |
|--------|-------|
| **Sprints completed** | 15 + QC audit + production hardening |
| **API routes** | 55 across 15 categories |
| **Database tables** | 16 with 62 RLS policies |
| **Migrations** | 23 (all synced) |
| **Test cases** | 1,380+ (unit, integration, RLS, E2E) |
| **Security audit** | 8 phases, 0 CRITICAL/HIGH findings remaining |
| **Auth coverage** | 99.25% statements, 100% functions |
| **Lighthouse** | 97 / 100 / 100 / 100 |
| **Compliance** | IRS Publication 4557 + OWASP Top 10 |

---

## What It Does

End-to-end tax practice management platform that handles the full client lifecycle: onboarding, document collection, appointment scheduling, payment processing, communication, and admin operations — all in one place. Designed so a solo IRS Enrolled Agent can manage 500+ clients without a technical team.

**Key capabilities:**
- Client portal with secure document upload, appointment booking, and payment history
- Admin portal with Kanban workflow, analytics, audit logs, campaign management, and user administration
- Two-factor authentication (email OTP + backup codes)
- Encrypted document storage with presigned URLs
- Square payment processing with idempotent transactions
- SMS and email campaign management
- Automated appointment and tax deadline reminders
- Dynamic service pricing management

---

## System Architecture

```
                     Internet (HTTPS)
                           |
                    +------v------+
                    |  Cloudflare  |  DDoS, WAF, rate limiting
                    +------+------+
                           |
              +------------+------------+
              |                         |
       +------v------+          +------v------+
       |   Vercel     |          |   Docker    |  Self-hosted path
       |  (Edge SSR)  |          | (Container) |
       +------+------+          +------+------+
              |                         |
              +-----------+-------------+
                          |
         +----------------+-----------------+
         |                |                 |
  +------v------+  +------v------+  +-------v-------+
  |   Supabase  |  | Backblaze B2|  |   Services    |
  |  PostgreSQL |  |  AES-256    |  |               |
  |  + RLS      |  |  Encrypted  |  | Square (pay)  |
  |  + Realtime |  |  Storage    |  | Twilio (SMS)  |
  +--------------+ +--------------+ | SMTP (email)  |
                                    | Redis (cache) |
                                    +---------------+
```

**Dual deployment:** Vercel edge for speed, Docker container for self-hosted control. Both share the same codebase — environment variables determine the deployment target.

---

## Tech Stack

| Layer | Technology | Why |
|-------|-----------|-----|
| **Frontend** | Next.js (App Router), TypeScript strict | SSR for SEO, full-stack in one codebase |
| **UI** | Tailwind CSS, shadcn/ui, Radix primitives | Accessible components, consistent design system |
| **State** | Zustand + TanStack Query | Simple global state + server cache with deduplication |
| **Database** | Supabase PostgreSQL | Managed Postgres with RLS, realtime subscriptions |
| **Auth** | Custom JWT sessions (jose) + bcryptjs | Stateless auth with httpOnly cookies, no third-party session dependency |
| **2FA** | Email OTP + encrypted backup codes | TOTP-style flow without requiring an authenticator app |
| **File Storage** | Backblaze B2 (S3-compatible) | AES-256 encryption, presigned URLs, cost-effective |
| **Payments** | Square SDK | Checkout, invoicing, idempotent transactions |
| **Email** | SMTP via nodemailer + React Email templates | Transactional emails, campaign sends, OTP delivery |
| **SMS** | Twilio | Appointment reminders, deadline alerts, campaigns |
| **Rate Limiting** | Redis-backed with in-memory fallback | Per-route limits, graceful degradation |
| **Deployment** | Vercel + Docker | Edge SSR or self-hosted container |
| **CDN/Security** | Cloudflare | DDoS protection, WAF, caching |
| **Testing** | Vitest + Playwright | Unit/integration/RLS + E2E security tests |

---

## Features Shipped (15 Sprints)

### Sprint 1: Foundation
- Next.js App Router with TypeScript strict mode
- Supabase client/server configuration with type generation
- Tailwind + shadcn/ui design system
- Deployment pipeline

### Sprint 2: Marketing Site
- SEO-optimized landing pages (Home, Services, About, Contact)
- Service pricing cards with dynamic content
- Structured data (JSON-LD) for local search
- Contact form with rate limiting

### Sprint 3: Authentication
- Email/password registration with Zod validation
- Custom JWT session management (jose, httpOnly cookies)
- Protected routes via middleware
- Role-based access (admin vs. client)
- Row-Level Security policies on all tables

### Sprint 4: Appointment Scheduling
- Interactive booking modal with calendar UI
- Availability management (admin controls open slots)
- Email confirmations
- Appointment status workflow (scheduled -> completed/cancelled/no-show)

### Sprint 5: Secure Document Management
- Drag-and-drop file upload with progress indicators
- S3-compatible storage integration with AES-256 server-side encryption
- Presigned URLs with 1-hour expiration (no public access)
- Document categorization (W-2, 1099, receipts, returns, 14 categories)
- Tax year organization and metadata tracking

### Sprint 6: Payment Processing
- Square Web Payments SDK checkout flow
- Service-based pricing with configurable tiers
- Idempotent payment creation (no double-charges)
- Payment receipt generation and email delivery
- Refund handling with audit trail

### Sprint 7: Admin Dashboard & Notifications
- Client Kanban board (onboarding -> active -> WIP -> completed)
- SMS appointment reminders via Twilio
- Email receipt system
- Client status tracking with notes
- Admin-only views with RLS enforcement

### Sprint 8: Communications & Engagement
- SMS campaign management (bulk sends, templates)
- Email campaign system with React Email templates
- Client engagement tracking and analytics

### Sprint 9: Polish & Code Quality
- Comprehensive error boundaries and fallback UI
- Loading skeletons across all pages
- Form validation improvements (real-time feedback)
- Code quality sweep (lint, dead code removal, consistency)

### Sprint 10: Content & Copy
- Professional copy for all marketing pages
- Service descriptions aligned with business offerings
- SEO metadata and Open Graph tags
- Accessibility audit and ARIA improvements

### Sprint 11: UI/UX Redesign
- Visual refresh across client and admin portals
- Mobile-responsive layout improvements
- Navigation and information architecture updates
- Consistent spacing, typography, and color system

### Sprint 12: Admin & Email
- Email template system for transactional messages
- Admin settings panel
- Notification preferences (per-user email/SMS opt-in)
- Cron-based automated reminders (appointments + tax deadlines)

### Sprint 13: Admin Panel Completion
- Lead management with conversion workflow
- Analytics dashboard (revenue, clients, appointments, trends)
- Audit log viewer with filtering and search
- User management (view, edit, deactivate accounts)
- Active session management (view/revoke)

### Sprint 14: Headshot Upload & RLS Fix
- Admin headshot upload with image processing
- Storage integration for profile images
- RLS policy corrections for edge cases
- Database migration cleanup

### Sprint 15: Admin Enhancements & Auth Fix
- Dynamic pricing management (CRUD for service tiers)
- Auth flow bug fixes (session edge cases, redirect logic)
- Admin dashboard status widgets
- Login history and session tracking

### Post-Sprint: Security Hardening
- Two-factor authentication (email OTP + backup codes)
- Content Security Policy headers
- Information leak remediation (password reset, error messages)
- Input validation hardening (Zod schemas on all 55 routes)
- Dependency audit and cleanup
- CSRF protection on state-changing operations

---

## Admin Portal

15 admin pages providing full practice management:

| Page | Purpose |
|------|---------|
| **Dashboard** | KPI widgets, status overview, quick actions |
| **Clients** | Kanban board, client lifecycle management |
| **Client Detail** | Individual client view with docs, payments, notes |
| **Leads** | Inbound lead tracking with conversion workflow |
| **Appointments** | Schedule management with detail views |
| **Payments** | Transaction history, refund management |
| **Analytics** | Revenue trends, client metrics, appointment stats |
| **Audit Log** | Timestamped action log with search and filtering |
| **Users** | Account management, role assignment, deactivation |
| **Sessions** | Active session viewer with revocation |
| **Campaigns (SMS)** | Bulk SMS sends with template management |
| **Campaigns (Email)** | Email campaigns with React Email templates |
| **Pricing** | Dynamic service tier management (CRUD) |
| **Settings** | App configuration, integrations, preferences |
| **Usage** | Platform usage metrics and resource tracking |

---

## Database Design

**16 tables**, 23 migrations, 62 RLS policies. Key design decisions:

- **UUID primary keys** everywhere (no sequential IDs exposed)
- **JSONB columns** for flexible metadata (addresses, preferences)
- **Separate status tracking table** for audit trail on client lifecycle changes
- **Document metadata only in DB** — actual files in encrypted external storage
- **RLS everywhere**: Clients see only their own data. Admin sees everything. Service-role only for sensitive tokens.

```
users ──────────── profiles
  |                     |
  |──── appointments    |──── documents (metadata -> encrypted storage)
  |──── payments        |──── tax_returns
  |──── messages        |──── sms_log
  |──── client_status (audit trail)
  |──── otp_codes (2FA)
  |──── backup_codes (2FA recovery)
  |
  |──── password_reset_tokens    (service-role only)
  |──── email_verification_tokens (service-role only)
  |──── audit_logs               (service-role only)
  |
pricing_tiers (public read, admin write)
leads (anonymous insert, admin manage)
app_settings (admin only)
newsletter_subscribers (separate, no user link)
```

### Tables by Access Pattern

| Pattern | Tables |
|---------|--------|
| User CRUD own data | users, profiles, appointments, documents |
| User read own + admin CRUD | payments, sms_log |
| Admin only | app_settings, client_status |
| Service role only | password_reset_tokens, email_verification_tokens, audit_logs, otp_codes, backup_codes |
| Public read, admin CRUD | pricing_tiers |
| Anonymous insert | leads |

---

## Security & Compliance

### 8-Phase Security Audit (Gate 16 — Passed)

| Phase | Scope | Result |
|-------|-------|--------|
| **Phase 0** | Critical findings remediation | 4/4 HIGH findings fixed |
| **Phase 1** | Static analysis | 92/100 score |
| **Phase 2** | Auth unit tests | 99.25% statement coverage, 100% functions (9/9 files) |
| **Phase 3** | API route auth tests | 648 tests passing, 0 failures |
| **Phase 4** | RLS policy verification | 73/73 checks pass, 20/20 integration tests |
| **Phase 5** | E2E security tests | 47/47 tests across 9 categories |
| **Phase 6** | Penetration testing | 0 CRITICAL, 0 HIGH remaining (1 LOW: infrastructure) |
| **Phase 7** | Compliance validation | 59/59 checks (22 IRS Pub 4557 + 37 OWASP Top 10) |

### IRS Publication 4557 Compliance

| Requirement | Implementation |
|-------------|---------------|
| **Encryption at rest** | AES-256 server-side encryption on all stored documents |
| **Encryption in transit** | TLS 1.3 enforced (Cloudflare + application) |
| **Access control** | Row-Level Security on all 16 tables (62 policies) |
| **Document retention** | 7-year lifecycle policies on storage buckets |
| **Presigned URLs** | 1-hour max expiration, no public file access |
| **Audit logging** | All state changes tracked with timestamps and actor |
| **Rate limiting** | Redis-backed per-route middleware with fallback |
| **Input validation** | Zod schemas on all 55 API routes, server-side validation |
| **2FA** | Email OTP with encrypted backup codes for account recovery |

### OWASP Top 10 Coverage

All 10 categories validated with automated and manual testing:

- **A01 Broken Access Control** — RLS + middleware auth checks on every route
- **A02 Cryptographic Failures** — AES-256 at rest, TLS 1.3 in transit, bcryptjs for passwords
- **A03 Injection** — Parameterized queries (Supabase SDK), Zod input validation
- **A04 Insecure Design** — Threat modeling per feature, defense in depth
- **A05 Security Misconfiguration** — CSP headers, secure cookie flags, no default credentials
- **A06 Vulnerable Components** — Dependency audit, single bcrypt implementation
- **A07 Auth Failures** — Custom JWT with httpOnly cookies, 2FA, rate-limited login
- **A08 Data Integrity** — Idempotent payments, CSRF protection, signed tokens
- **A09 Logging Failures** — Comprehensive audit log, security event tracking
- **A10 SSRF** — No user-controlled URLs in server-side requests

---

## API Routes

55 routes across 15 categories:

```
/api/
  auth/           login, register, logout, verify, forgot-password,
                  reset-password, resend-verification
  auth/2fa/       enable, disable, verify, backup-codes
  profile/        update, change-password, login-history, notifications
  appointments/   book, cancel, my-appointments, next-available,
                  availability, send-confirmation
  documents/      upload, list, download, delete
  payments/       create, list, refund
  contact/        public contact form
  leads/          public lead capture
  pricing/        public pricing display
  headshot/       profile image management
  health/         application health check
  receipts/       email receipt delivery
  settings/       integration configuration, connection testing
  cron/           appointment-reminders, tax-deadline-reminders
  admin/          dashboard, status, clients, leads, users,
                  sessions, appointments, payments, documents,
                  analytics, audit, pricing, campaigns (sms/email),
                  settings (headshot), lead-conversion
```

---

## Quality

| Metric | Score |
|--------|-------|
| **Lighthouse Performance** | 97 |
| **Lighthouse Accessibility** | 100 |
| **Lighthouse Best Practices** | 100 |
| **Lighthouse SEO** | 100 |
| **TypeScript** | Strict mode, zero `any` in production code |
| **Test coverage** | 1,380+ test cases across 59 test files |
| **Error handling** | Error boundaries on all route segments, fallback UI |
| **Security headers** | CSP, X-Frame-Options, X-Content-Type-Options, Referrer-Policy |

---

## What I'd Do Differently

- **Start with E2E tests earlier** — Playwright was added in Sprint 7, should have been Sprint 1. The security E2E suite caught real bugs that unit tests missed.
- **Custom auth from day 1** — Started with Supabase Auth, migrated to custom JWT sessions mid-project. The migration was clean but added a sprint of rework.
- **Component library from day 1** — Some early components were rebuilt when shadcn patterns were adopted. Establishing the design system in Sprint 1 would have saved iteration.
- **Separate admin from client earlier** — The admin portal grew to 15 pages. Starting with a dedicated admin layout in Sprint 1 would have simplified routing.
- **Redis for sessions sooner** — In-memory rate limiting works but doesn't survive restarts. Redis integration should have been Sprint 3, not post-QC.

---

## Roadmap (Gate 17: Launch)

- [ ] Domain cutover and SSL configuration
- [ ] Email deliverability setup (SPF, DKIM, DMARC)
- [ ] Beta testing with 5-10 real clients
- [ ] Cross-browser and mobile QA pass
- [ ] Analytics and uptime monitoring
- [ ] Square production credentials
- [ ] Client onboarding documentation

---

## Why Source Code Is Private

This is a **paid client project**. The business logic, client data schemas, and integrations are proprietary. This showcase exists so recruiters and collaborators can evaluate the architecture, security posture, and technical decisions without accessing the codebase.

If you're evaluating my work, the best way to see it in action is to visit [herakles.dev](https://herakles.dev) or reach out at [hello@herakles.dev](mailto:hello@herakles.dev).

---

**Built by** [D. Michael Piscitelli](https://github.com/herakles-dev) — Full-stack engineer, Chicago IL
