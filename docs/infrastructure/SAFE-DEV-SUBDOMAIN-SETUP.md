# Safe Dev-V26 Subdomain Setup Guide

**Date:** 2026-04-21 (Updated)  
**Risk Level:** Low (with proper backups)  
**Rollback Time:** < 5 minutes  

---

## Current State Analysis

### Production VPS: 40.90.168.38

| Domain | Status | Purpose |
|--------|--------|---------|
| app.eventxgames.com | Production | Production frontend |
| app-api.eventxgames.com | Production | Production API |

### Staging VPS (NEW): 172.188.98.210

| Domain | Status | Purpose |
|--------|--------|---------|
| dev-app-v26.eventxgames.com | DNS Ready | Dev frontend |
| dev-api-v26.eventxgames.com | DNS Ready | Dev API |
| dev-ws-v26.eventxgames.com | DNS Ready | Dev WebSocket |

### What We Want to Add

| New Domain | Purpose | Risk |
|------------|---------|------|
| dev-app-v26.eventxgames.com | Dev frontend | Low |
| dev-api-v26.eventxgames.com | Dev API | Low |
| dev-ws-v26.eventxgames.com | Dev WebSocket | Low |

---

## IMPORTANT: Backup First!

Before making ANY changes, create backups:

```bash
# SSH to VPS
ssh root@172.188.98.210

# Create timestamped backups
BACKUP_DATE=$(date +%Y%m%d_%H%M%S)
cp /root/src/Caddyfile /root/src/Caddyfile.backup.$BACKUP_DATE
cp /root/src/.env /root/src/.env.backup.$BACKUP_DATE
cp /root/src/docker-compose.yml /root/src/docker-compose.yml.backup.$BACKUP_DATE

# Verify backups exist
ls -la /root/src/*.backup.*
```

---

## Step-by-Step Guide

### Step 1: Add DNS Records in Azure Portal

**Who:** CTO or someone with Azure Portal access

1. **Login to Azure Portal**
   - Go to: https://portal.azure.com

2. **Navigate to DNS Zone**
   - Search for "DNS zones" in the search bar
   - Click on "eventxgames.com" zone

3. **Add A Records**

   Click "+ Record set" and add each:

   | Name | Type | TTL | Value |
   |------|------|-----|-------|
   | dev-app-v26 | A | 300 | 172.188.98.210 |
   | dev-api-v26 | A | 300 | 172.188.98.210 |
   | dev-ws-v26 | A | 300 | 172.188.98.210 |

   **TTL 300** = 5 minutes (for quick propagation during testing)

   > **Note:** DNS is already configured and resolving correctly.

4. **Verify DNS Propagation**

   Wait 5-10 minutes, then test:
   ```bash
   nslookup dev-app-v26.eventxgames.com
   nslookup dev-api-v26.eventxgames.com
   ```

   Should return: 172.188.98.210

---

### Step 2: Update Caddyfile (SAFELY)

**Who:** Engineer with VPS access

```bash
# SSH to VPS
ssh root@172.188.98.210

# Go to project directory
cd /root/src

# FIRST: Make sure backup exists
ls -la Caddyfile.backup.*

# Edit Caddyfile using nano
nano /root/src/Caddyfile
```

**Add this to the END of the file** (do not modify existing config):

```caddyfile
# ============================================
# DEV ENVIRONMENT - V26
# Added: 2026-04-16
# Safe to remove if issues occur
# ============================================

# Dev Frontend
dev-app-v26.eventxgames.com {
    reverse_proxy frontend:3000 {
        header_up Host {host}
        header_up X-Real-IP {remote}
        header_up X-Forwarded-For {remote}
        header_up X-Forwarded-Proto {scheme}
    }
    header {
        X-Content-Type-Options nosniff
        X-Frame-Options DENY
        Referrer-Policy strict-origin-when-cross-origin
        -Server
    }
    encode gzip
}

# Dev API
dev-api-v26.eventxgames.com {
    reverse_proxy backend:3001 {
        header_up Host {host}
        header_up X-Real-IP {remote}
        header_up X-Forwarded-For {remote}
        header_up X-Forwarded-Proto {scheme}
    }
    header {
        Access-Control-Allow-Origin https://dev-app-v26.eventxgames.com
        Access-Control-Allow-Methods "GET, POST, PUT, PATCH, DELETE, OPTIONS"
        Access-Control-Allow-Headers "Content-Type, Authorization"
        X-Content-Type-Options nosniff
        -Server
    }
    encode gzip
}

# Dev WebSocket
dev-ws-v26.eventxgames.com {
    reverse_proxy backend:3001 {
        header_up Host {host}
        header_up X-Real-IP {remote}
        header_up X-Forwarded-For {remote}
        header_up X-Forwarded-Proto {scheme}
        header_up Connection {http.request.header.Connection}
        header_up Upgrade {http.request.header.Upgrade}
    }
}
```

---

### Step 3: Update CORS Settings (SAFELY)

```bash
# SSH to VPS
ssh root@172.188.98.210

# Check current CORS setting
grep ALLOWED_ORIGINS /root/src/.env

# Edit .env file
nano /root/src/.env

# Find the ALLOWED_ORIGINS line and ADD dev domain:
# BEFORE: ALLOWED_ORIGINS=http://staging.eventxgames.com,https://staging.eventxgames.com,http://localhost:3000
# AFTER:  ALLOWED_ORIGINS=http://staging.eventxgames.com,https://staging.eventxgames.com,http://localhost:3000,https://dev-app-v26.eventxgames.com

# Save and exit (Ctrl+X, Y, Enter)
```

---

### Step 4: Test Configuration Before Restart

```bash
# SSH to VPS
ssh root@172.188.98.210

# Validate Caddyfile syntax (inside container)
docker exec eventxgames-caddy caddy validate --config /etc/caddy/Caddyfile

# If validation fails, restore backup:
# cp /root/src/Caddyfile.backup.TIMESTAMP /root/src/Caddyfile
```

---

### Step 5: Restart Services (One at a Time)

```bash
# SSH to VPS
ssh root@172.188.98.210
cd /root/src

# Restart Caddy ONLY first
docker compose restart caddy

# Wait 30 seconds and check
sleep 30
docker ps | grep caddy

# If Caddy is healthy, restart backend
docker compose restart backend

# Wait and verify
sleep 10
docker ps | grep backend
```

---

### Step 6: Verify Everything Works

```bash
# Test staging (should still work)
curl -I https://staging.eventxgames.com
curl -I https://staging-api.eventxgames.com/health

# Test new dev subdomains
curl -I https://dev-app-v26.eventxgames.com
curl -I https://dev-api-v26.eventxgames.com/health
```

---

## ROLLBACK PROCEDURE (If Something Breaks)

### Quick Rollback (< 2 minutes)

```bash
# SSH to VPS
ssh root@172.188.98.210
cd /root/src

# Find the most recent backup
ls -la *.backup.*

# Restore Caddyfile (replace TIMESTAMP with actual)
cp Caddyfile.backup.TIMESTAMP Caddyfile

# Restore .env
cp .env.backup.TIMESTAMP .env

# Restart services
docker compose restart caddy backend

# Verify staging works
curl -I https://staging.eventxgames.com
```

### Remove Only Dev Config (Alternative)

If you want to remove just the dev config without full rollback:

1. Edit Caddyfile: `nano /root/src/Caddyfile`
2. Delete everything from `# DEV ENVIRONMENT - V26` to the end
3. Edit .env: Remove `,https://dev-app-v26.eventxgames.com` from ALLOWED_ORIGINS
4. Restart: `docker compose restart caddy backend`

---

## Checklist for CTO

### Before Starting

- [ ] Backup confirmation received from engineer
- [ ] Azure Portal access verified
- [ ] VPS SSH access verified
- [ ] Understand rollback procedure

### DNS Setup (Azure Portal)

- [x] Added A record: dev-app-v26 -> 172.188.98.210
- [x] Added A record: dev-api-v26 -> 172.188.98.210
- [x] Added A record: dev-ws-v26 -> 172.188.98.210
- [ ] Verified DNS propagation (nslookup)

### Server Configuration

- [ ] Caddyfile backed up
- [ ] Caddyfile updated with dev domains
- [ ] .env backed up
- [ ] CORS updated with dev domains
- [ ] Configuration validated
- [ ] Caddy restarted successfully
- [ ] Backend restarted successfully

### Verification

- [ ] staging.eventxgames.com still works
- [ ] staging-api.eventxgames.com still works
- [ ] dev-app-v26.eventxgames.com accessible
- [ ] dev-api-v26.eventxgames.com/health returns OK
- [ ] SSL certificates provisioned (HTTPS working)

---

## Timeline Estimate

| Step | Duration | Who |
|------|----------|-----|
| Backups | 2 minutes | Engineer |
| DNS setup | 5 minutes | CTO |
| DNS propagation | 5-10 minutes | Wait |
| Caddyfile update | 5 minutes | Engineer |
| CORS update | 2 minutes | Engineer |
| Restart services | 2 minutes | Engineer |
| Verification | 5 minutes | Engineer |
| **Total** | **~25-30 minutes** | |

---

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Staging breaks | Low | High | Backups + quick rollback |
| DNS misconfiguration | Low | Low | TTL=300 allows fast fix |
| SSL cert failure | Low | Low | Caddy auto-retries |
| CORS issues | Medium | Low | Easy to fix in .env |

---

*Document created: 2026-04-16*  
*Updated: 2026-04-21 - Corrected IP to new staging VPS (172.188.98.210)*  
*Safe to share with CTO and engineering team*
