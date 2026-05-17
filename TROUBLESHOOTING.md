# ENIAC Systems - Troubleshooting Guide

> Common issues and solutions for all ENIAC Systems projects.

---

## Table of Contents

1. [Deployment Issues](#deployment-issues)
2. [Database Issues](#database-issues)
3. [SSO & Authentication](#sso--authentication)
4. [CI/CD Pipeline](#cicd-pipeline)
5. [Application Errors](#application-errors)
6. [Performance Issues](#performance-issues)
7. [FTP/Hosting Issues](#ftphosting-issues)
8. [Shared-Lib Issues](#shared-lib-issues)

---

## Deployment Issues

### Deploy Fails with FTPS Error

**Symptom:** GitHub Actions workflow fails at "Upload via FTPS" step.

**Causes & Solutions:**

1. **FTP credentials expired**
   - Verify FTP credentials in GitHub repository secrets
   - Update: Settings → Secrets and variables → Actions
   - Required secrets: `FTP_HOST`, `FTP_USERNAME`, `FTP_PASSWORD`, `FTP_PORT`

2. **FTPS connection timeout**
   - HostGator FTPS may timeout during large uploads
   - Solution: Retry the workflow (transient issue)
   - If persistent, check HostGator server status

3. **Permission denied on remote directory**
   - Verify the target directory exists on the server
   - Check FTP user has write permissions to `/public_html/app/{project}/`

### Deploy Succeeds but Site Shows Old Version

**Symptom:** Deploy reports success but changes are not visible.

**Solutions:**

1. **Browser cache** - Hard refresh (Ctrl+F5) or clear cache
2. **OPcache** - PHP OPcache may hold old bytecode
   - Add `?opcache_reset` to any PHP file URL if accessible
   - Or wait for OPcache to expire (default: 2 hours)
3. **Cloudflare cache** - Purge Cloudflare cache from dashboard

### Missing Assets After Deploy

**Symptom:** CSS/JS files return 404 after deployment.

**Solution:**
```bash
# Ensure assets are built before deploy
npm run build
# Or for Laravel Mix
npm run production
```

Verify `.github/workflows/deploy.yml` includes asset compilation step.

---

## Database Issues

### Migration Fails

**Symptom:** `php artisan migrate` throws error.

**Common Causes:**

1. **Foreign key constraint failure**
   - Check migration order (dependencies first)
   - Use `Schema::disableForeignKeyConstraints()` if needed

2. **Table already exists**
   - Database may have residual tables from previous install
   - Check with: `SHOW TABLES;`
   - Drop conflicting tables or use `--force` flag

3. **Connection refused**
   - Verify `.env` database credentials
   - Check database server is running
   - Verify database user has correct permissions

### Database Connection Timeout

**Symptom:** "SQLSTATE[HY000] [2002] Connection timed out"

**Solutions:**

1. Check HostGator database server status
2. Verify `DB_HOST` in `.env` (may need to use specific host, not `localhost`)
3. Check firewall rules allow connections from web server
4. Increase connection timeout in `config/database.php`

### Data Not Persisting

**Symptom:** Form submissions appear to work but data is lost.

**Check:**

1. Database transaction not committed
2. Wrong database connection in `.env`
3. Model using wrong table name (`protected $table`)
4. Soft deletes hiding records (`protected $casts`)

---

## SSO & Authentication

### SSO Not Working Between Projects

**Symptom:** User logs in to one project but not recognized in another.

**Troubleshooting:**

1. **Cookie domain mismatch**
   - Verify `SSO_DOMAIN=eniacsystems.com.br` in all projects
   - Cookie must be set for `.eniacsystems.com.br` (leading dot)

2. **Session fingerprint mismatch**
   - User agent or IP changed between requests
   - Check `SsoSessionManager` fingerprint logic

3. **shared-lib not updated**
   - Ensure all projects use the same shared-lib version
   - Run: `composer update eniacsystems/shared-lib`

4. **Cookie not being set**
   - Check browser dev tools → Application → Cookies
   - Verify cookie exists with correct domain and path

### Login Loop

**Symptom:** User enters credentials but is redirected back to login.

**Causes:**

1. CSRF token mismatch - Clear browser cache and cookies
2. Session storage full - Check `storage/framework/sessions/`
3. Database user table corrupted - Verify users table integrity
4. Rate limiting triggered - Wait for decay period (default: 15 min)

### 403 Forbidden on Admin Routes

**Symptom:** Accessing `/admin/*` returns 403.

**Solution:**

1. Verify user has admin role in database
2. Check middleware is correctly applied in routes
3. Verify SSO session is active
4. Check `SsoMiddleware` configuration

---

## CI/CD Pipeline

### PHPUnit Tests Fail

**Symptom:** `php vendor/bin/phpunit` returns failures.

**Steps:**

1. **Check test database configuration**
   - Tests may require separate test database
   - Verify `phpunit.xml` database settings

2. **Missing test dependencies**
   - Run: `composer install --dev`
   - Ensure test database is seeded

3. **Environment-specific failures**
   - Some tests may fail on different PHP versions
   - Check PHP version matches CI environment (8.2+)

### Pint Linting Fails

**Symptom:** `vendor/bin/pint --test` returns style violations.

**Solution:**
```bash
# Auto-fix style issues
vendor/bin/pint

# Then commit the changes
git add .
git commit -m "style: fix code style violations"
```

### Workflow Not Triggering

**Symptom:** Push to repository does not start CI workflow.

**Check:**

1. Workflow file exists in `.github/workflows/`
2. Workflow file has correct YAML syntax
3. Branch protection rules not blocking
4. GitHub Actions enabled for repository
5. Check Actions tab for any disabled workflows

---

## Application Errors

### 500 Internal Server Error

**Symptom:** Page returns HTTP 500.

**Troubleshooting:**

1. **Check Laravel logs**
   ```bash
   tail -f storage/logs/laravel.log
   ```

2. **Enable debug mode temporarily**
   ```env
   APP_DEBUG=true
   ```
   ⚠️ Disable immediately after debugging

3. **Common causes:**
   - Missing PHP extension (mbstring, fileinfo, etc.)
   - Incorrect file permissions
   - Missing `.env` or invalid `APP_KEY`
   - Database connection failure
   - Class not found (composer autoload issue)

### 404 Not Found

**Symptom:** Route returns 404.

**Check:**

1. Route exists in `routes/web.php` or `routes/api.php`
2. `.htaccess` is present in project root (for Apache)
3. URL matches route definition exactly
4. If subdirectory deployment, verify base URL configuration

### White Screen of Death

**Symptom:** Blank page with no output.

**Causes:**

1. PHP fatal error with display_errors off
2. Memory limit exceeded
3. Syntax error in PHP file

**Solution:**
```bash
# Check PHP error log
tail -f /var/log/php-fpm/error.log

# Or enable error display temporarily
ini_set('display_errors', 1);
error_reporting(E_ALL);
```

---

## Performance Issues

### Slow Page Loads

**Symptom:** Pages take >3 seconds to load.

**Optimizations:**

1. **Enable OPcache** (should be enabled on HostGator)
2. **Database query optimization**
   - Use eager loading: `User::with('posts')->get()`
   - Add indexes to frequently queried columns
   - Check for N+1 queries with Laravel Debugbar

3. **Asset optimization**
   - Minify CSS/JS
   - Use CDN for static assets
   - Enable browser caching headers

4. **Laravel optimization**
   ```bash
   php artisan config:cache
   php artisan route:cache
   php artisan view:cache
   php artisan optimize
   ```

### High Memory Usage

**Symptom:** PHP processes consuming excessive memory.

**Solutions:**

1. Check for memory leaks in long-running processes
2. Increase PHP memory limit if needed:
   ```ini
   memory_limit = 256M
   ```
3. Use chunking for large dataset operations:
   ```php
   User::chunk(100, function ($users) {
       // process
   });
   ```

---

## FTP/Hosting Issues

### HostGator Resource Limits

**Symptom:** Site goes down or shows errors during peak usage.

**HostGator Shared Hosting Limits:**

| Resource | Limit |
|----------|-------|
| CPU Usage | ~25% (shared) |
| Memory | 256MB per process |
| Entry Processes | ~25 concurrent |
| I/O Usage | ~1MB/s |

**Solutions:**

1. Optimize database queries to reduce CPU
2. Use caching to reduce database load
3. Consider upgrading to VPS if limits are consistently hit
4. Monitor usage in HostGator cPanel

### FTP Connection Refused

**Symptom:** Cannot connect via FTP/FTPS.

**Check:**

1. FTP credentials are correct
2. FTPS port (usually 21 or 990) is not blocked by firewall
3. HostGator FTP service is running
4. IP not blocked due to failed login attempts

### File Permission Issues

**Symptom:** "Permission denied" errors.

**Required Permissions:**

```bash
# Directories
chmod 755 storage/
chmod 755 bootstrap/cache/
chmod 755 public/uploads/

# Files
chmod 644 .env
chmod 644 storage/logs/*.log
```

---

## Shared-Lib Issues

### shared-lib Not Loading

**Symptom:** Class not found errors for SSO or validation classes.

**Solutions:**

1. **Composer autoload**
   ```bash
   composer dump-autoload
   ```

2. **Version mismatch**
   - Check `composer.json` requires correct shared-lib version
   - Update: `composer update eniacsystems/shared-lib`

3. **Repository not configured**
   - Verify VCS repository in `composer.json`:
   ```json
   "repositories": [
       {
           "type": "vcs",
           "url": "https://github.com/ENIACSystems/shared-lib.git"
       }
   ]
   ```

### Brazilian Validators Not Working

**Symptom:** CPF, CNPJ validation rules not recognized.

**Solution:**

1. Ensure shared-lib service provider is registered
2. Check `config/app.php` providers array
3. Run: `php artisan config:clear`

### SSO Middleware Not Applied

**Symptom:** Routes accessible without authentication.

**Check:**

1. Middleware registered in `app/Http/Kernel.php`
2. Routes wrapped in middleware group
3. `SsoMiddleware` returns correct response on failure

---

## Emergency Procedures

### Rollback a Failed Deploy

1. Go to GitHub Actions → Select failed workflow
2. Re-run previous successful workflow
3. Or manually upload previous version via FTP

### Database Recovery

1. Check HostGator backups (daily automatic)
2. Restore from cPanel → Backup → MySQL Restore
3. Contact HostGator support for point-in-time recovery

### Site Down - Immediate Actions

1. Check HostGator server status
2. Verify domain DNS is resolving
3. Check `.htaccess` not corrupted
4. Restore from backup if needed

---

## Contact & Support

| Channel | Details |
|---------|---------|
| Email | suporte@eniacsystems.com.br |
| GitHub Issues | [ENIACSystems/{Project}/issues](https://github.com/ENIACSystems) |
| Developer | Joao Aschenbrenner |

---

*Last updated: May 2026*
*Maintained by: ENIAC Systems Dev Team*
