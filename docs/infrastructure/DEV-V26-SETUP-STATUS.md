# Dev-V26 Subdomain Setup Status

**Date:** 2026-04-16  
**Status:** PREPARED (Waiting for DNS)

---

## Overview

Development environment subdomains for testing before production migration.

| Subdomain | Target | Status |
|-----------|--------|--------|
| dev-app-v26.eventxgames.com | Frontend (port 3000) | Config Ready |
| dev-api-v26.eventxgames.com | Backend API (port 3001) | Config Ready |
| dev-ws-v26.eventxgames.com | WebSocket (port 3001) | Config Ready |

---

## What's Done

### 1. Caddyfile Updated ✅

Location: `/root/src/Caddyfile`

Added configurations for:
- `dev-app-v26.eventxgames.com` → `frontend:3000`
- `dev-api-v26.eventxgames.com` → `backend:3001` (with CORS headers)
- `dev-ws-v26.eventxgames.com` → `backend:3001` (with WebSocket support)

### 2. CORS Updated ✅

Location: `/root/src/.env`

```
ALLOWED_ORIGINS=http://staging.eventxgames.com,https://staging.eventxgames.com,http://localhost:3000,https://dev-app-v26.eventxgames.com,http://dev-app-v26.eventxgames.com
```

### 3. Backups Created ✅

- `Caddyfile.backup.20260416_120242`
- `.env.backup.20260416_120242`

---

## What's Pending (Human Action Required)

### DNS Records in Azure Portal

Add these A records in Azure DNS Zone (eventxgames.com):

| Name | Type | TTL | Value |
|------|------|-----|-------|
| dev-app-v26 | A | 300 | 40.90.168.38 |
| dev-api-v26 | A | 300 | 40.90.168.38 |
| dev-ws-v26 | A | 300 | 40.90.168.38 |

**Instructions:**
1. Go to Azure Portal → DNS Zones → eventxgames.com
2. Click "+ Record set"
3. Add each record above
4. Wait 5-10 minutes for propagation

---

## After DNS is Added

Run these commands to activate:

```bash
# SSH to VPS
ssh root@40.90.168.38

# Restart Caddy to pick up new domains
docker compose -f /root/src/docker-compose.yml restart caddy

# Wait 30 seconds for SSL certificates
sleep 30

# Restart backend for CORS changes
docker compose -f /root/src/docker-compose.yml restart backend

# Verify
curl -I https://dev-app-v26.eventxgames.com
curl -I https://dev-api-v26.eventxgames.com/health
```

---

## Rollback Procedure

If issues occur:

```bash
# Restore backups
cd /root/src
cp Caddyfile.backup.20260416_120242 Caddyfile
cp .env.backup.20260416_120242 .env

# Restart
docker compose restart caddy backend
```

---

*Prepared by: Paperclip AI DevOps Agent*  
*Date: 2026-04-16*
