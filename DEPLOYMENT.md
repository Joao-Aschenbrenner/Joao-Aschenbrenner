# ENIAC Systems - Deployment Guide

> Master deployment documentation for all ENIAC Systems projects.

---

## Architecture Overview

ENIAC Systems operates a **multi-project SaaS ecosystem** hosted on shared infrastructure. Each project is an independent Laravel application deployed to a subdirectory under `eniacsystems.com.br/app/`.

### System Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    eniacsystems.com.br                       │
├─────────────────────────────────────────────────────────────┤
│  Landing Page (WordPress/Static)                             │
├─────────────────────────────────────────────────────────────┤
│  /app/                                                       │
│  ├── /mecanic-system/        → MecanicSystem                │
│  ├── /schospital-manager/    → SCHospitalManager            │
│  ├── /simple-finance/        → SimpleFinance                │
│  ├── /snap-select/           → SnapSelect                   │
│  ├── /rabbit/                → RABBIT                       │
│  ├── /bling-stock-sync/      → BlingStockSync               │
│  └── /audesp-v-tjson/        → AudespV-TJSON                │
├─────────────────────────────────────────────────────────────┤
│  /internal-system/           → InternalSystem (Admin Panel)  │
└─────────────────────────────────────────────────────────────┘
```

### Shared Components

| Component | Repository | Purpose |
|-----------|------------|---------|
| **shared-lib** | [ENIACSystems/shared-lib](https://github.com/ENIACSystems/shared-lib) | SSO, Security Headers, Brazilian Validators, Helpers |
| **InternalSystem** | [ENIACSystems/InternalSystem](https://github.com/ENIACSystems/InternalSystem) | Central admin panel, user management, project monitoring |

---

## Project Inventory

| Project | Repository | Stack | Production URL |
|---------|------------|-------|----------------|
| MecanicSystem | [ENIACSystems/MecanicSystem](https://github.com/ENIACSystems/MecanicSystem) | Laravel 10 | [/app/mecanic-system](https://eniacsystems.com.br/app/mecanic-system/) |
| SCHospitalManager | [ENIACSystems/SCHospitalManager](https://github.com/ENIACSystems/SCHospitalManager) | Laravel 10 | [/app/schospital-manager](https://eniacsystems.com.br/app/schospital-manager/) |
| SimpleFinance | [ENIACSystems/SimpleFinance](https://github.com/ENIACSystems/SimpleFinance) | Laravel 10 | [/app/simple-finance](https://eniacsystems.com.br/app/simple-finance/) |
| SnapSelect | [ENIACSystems/SnapSelect](https://github.com/ENIACSystems/SnapSelect) | Laravel 11 | [/app/snap-select](https://eniacsystems.com.br/app/snap-select/) |
| RABBIT | [ENIACSystems/RABBIT](https://github.com/ENIACSystems/RABBIT) | Laravel 10 | [/app/rabbit](https://eniacsystems.com.br/app/rabbit/) |
| BlingStockSync | [ENIACSystems/BlingStockSync](https://github.com/ENIACSystems/BlingStockSync) | Laravel 10 | [/app/bling-stock-sync](https://eniacsystems.com.br/app/bling-stock-sync/) |
| AudespV-TJSON | [ENIACSystems/AudespV-TJSON](https://github.com/ENIACSystems/AudespV-TJSON) | Laravel 10 | [/app/audesp-v-tjson](https://eniacsystems.com.br/app/audesp-v-tjson/) |
| ENIAC-SENTINEL | [ENIACSystems/ENIAC-SENTINEL](https://github.com/ENIACSystems/ENIAC-SENTINEL) | Laravel 10 | In Development |
| shared-lib | [ENIACSystems/shared-lib](https://github.com/ENIACSystems/shared-lib) | PHP Package | N/A (dependency) |
| internal-system-core | [ENIACSystems/internal-system-core](https://github.com/ENIACSystems/internal-system-core) | PHP MVC | Archived |
| InternalSystem | [ENIACSystems/InternalSystem](https://github.com/ENIACSystems/InternalSystem) | PHP Custom | [/internal-system](https://eniacsystems.com.br/internal-system/) |

---

## CI/CD Pipeline

### GitHub Actions Workflow

Each project uses GitHub Actions for CI/CD with the following stages:

```
Push/PR → Lint (Pint) → Test (PHPUnit) → Build → Deploy (FTPS)
```

### Workflow Files

| File | Purpose |
|------|---------|
| `.github/workflows/ci.yml` | Continuous Integration (lint + test) |
| `.github/workflows/deploy.yml` | Continuous Deployment (manual trigger) |

### Deploy Process

1. **Trigger:** Manual `workflow_dispatch` from GitHub Actions tab
2. **Build:** Composer install (production), asset compilation
3. **Transfer:** FTPS upload to HostGator shared hosting
4. **Target:** `/public_html/app/{project-slug}/`

### Deploy Commands

```bash
# Trigger deploy via GitHub CLI
gh workflow run deploy.yml -R ENIACSystems/{ProjectName}

# Or use the PowerShell deploy script
./deploy-ftp.ps1 -Project {ProjectName}
```

---

## Environment Configuration

### Required Environment Variables

Each Laravel project requires a `.env` file with:

```env
APP_NAME="Project Name"
APP_ENV=production
APP_DEBUG=false
APP_URL=https://eniacsystems.com.br/app/{project-slug}
APP_KEY=base64:xxxxxxxxxxxxxxxxxxxxxxxxxxxxx

DB_CONNECTION=mysql
DB_HOST={host}
DB_PORT=3306
DB_DATABASE=eniacs42_{project_db}
DB_USERNAME={user}
DB_PASSWORD={password}

# SSO Configuration (shared across projects)
SSO_DOMAIN=eniacsystems.com.br
SSO_COOKIE_NAME=eniac_session

# SMTP (optional)
MAIL_MAILER=smtp
MAIL_HOST=smtp.hostgator.com.br
MAIL_PORT=587
MAIL_USERNAME=adm@eniacsystems.com.br
MAIL_PASSWORD={password}
MAIL_ENCRYPTION=tls
```

### Database Naming Convention

All databases follow the pattern: `eniacs42_{project_identifier}`

| Project | Database Name |
|---------|---------------|
| MecanicSystem | eniacs42_mecanic_system |
| SCHospitalManager | eniacs42_schospital |
| SimpleFinance | eniacs42_simplefinance |
| SnapSelect | eniacs42_snapselect |
| RABBIT | eniacs42_rabbit |
| BlingStockSync | eniacs42_blingstock |
| AudespV-TJSON | eniacs42_audesp |

---

## SSO Architecture

All projects share a unified Single Sign-On system via `shared-lib`:

```
User logs in → SSO cookie set (.eniacsystems.com.br)
             → All projects validate cookie via SsoMiddleware
             → Session fingerprint prevents hijacking
```

### SSO Flow

1. User authenticates at any project or InternalSystem
2. `SsoSessionManager` creates wildcard cookie
3. Other projects detect cookie via `SsoMiddleware`
4. Session validated against central user database
5. User auto-logged in without re-authentication

---

## Monitoring

### Stats Endpoint

Each project exposes an admin API endpoint for centralized monitoring:

```
GET /admin-api/stats
```

Returns:
- User count
- Active sessions
- Database size
- Last deploy timestamp
- Error count (7-day window)

### Admin Panel

InternalSystem aggregates stats from all projects:
- URL: `https://eniacsystems.com.br/internal-system/AdminManager/`
- Dashboard shows all project health metrics

---

## Security

### Applied Measures

| Measure | Implementation |
|---------|----------------|
| SQL Injection | Prepared statements (Eloquent) |
| XSS | Output escaping via Blade |
| CSRF | Token validation on all mutations |
| Rate Limiting | Login brute-force protection |
| Security Headers | HSTS, X-Frame-Options, CSP |
| Session Security | HttpOnly, Secure, SameSite cookies |
| File Upload | MIME type validation, whitelist |
| SSO Fingerprint | Device fingerprinting prevents hijacking |

### Compliance

- LGPD (Brazilian GDPR) compliant
- OWASP Top 10 mitigations applied
- Audit logging on all critical operations

---

## Hosting Infrastructure

| Component | Provider | Details |
|-----------|----------|---------|
| Web Hosting | HostGator | Shared hosting (Linux) |
| Database | HostGator | MySQL 8.0 |
| CDN/Domain | Cloudflare | DNS + SSL |
| CI/CD | GitHub Actions | Free tier |
| Deploy Protocol | FTPS | Encrypted file transfer |

---

## Quick Reference

### Repository URLs

- **Organization:** [github.com/ENIACSystems](https://github.com/ENIACSystems)
- **Profile:** [github.com/Joao-Aschenbrenner](https://github.com/Joao-Aschenbrenner)
- **Portfolio:** [eniacsystems.com.br](https://eniacsystems.com.br/)

### Deploy Checklist

- [ ] Tests pass locally (`php vendor/bin/phpunit`)
- [ ] Lint passes (`vendor/bin/pint --test`)
- [ ] `.env` configured for production
- [ ] `APP_DEBUG=false`
- [ ] Migrations tested
- [ ] Trigger workflow_dispatch on GitHub
- [ ] Verify deployment on production URL
- [ ] Check /admin-api/stats endpoint

---

*Last updated: May 2026*
*Maintained by: ENIAC Systems Dev Team*
