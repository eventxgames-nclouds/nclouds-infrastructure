# Task 001: Set Up Dev-V26 Subdomains

**Status:** Pending  
**Priority:** High  
**Assigned To:** Paperclip Agent (Claude)  
**Created:** 2026-04-16  

---

## Objective

Set up development subdomains for testing before production deployment:

| Subdomain | Purpose | Target |
|-----------|---------|--------|
| dev-app-v26.eventxgames.com | Frontend testing | Port 3000 |
| dev-api-v26.eventxgames.com | Backend API testing | Port 3001 |
| dev-ws-v26.eventxgames.com | WebSocket testing | Port 3001/ws |

---

## Prerequisites

- [ ] DNS records added (Ask senior/CTO)
- [ ] VPS access confirmed
- [ ] Current Caddyfile backed up

---

## Steps

### Step 1: Backup Current Caddyfile

```bash
ssh root@40.90.168.38
cp /root/src/Caddyfile /root/src/Caddyfile.backup.$(date +%Y%m%d)
```

### Step 2: Add Dev Subdomain Configuration

Add to /root/src/Caddyfile:

```caddyfile
# DEV Environment - v26
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

### Step 3: Update Backend CORS

In /root/src/backend/.env or docker-compose.yml, add:

```
ALLOWED_ORIGINS=https://app.eventxgames.com,https://staging.eventxgames.com,https://dev-app-v26.eventxgames.com
```

### Step 4: Restart Services

```bash
cd /root/src
docker compose restart caddy
docker compose restart backend
```

### Step 5: Verify

```bash
curl -I https://dev-app-v26.eventxgames.com
curl -I https://dev-api-v26.eventxgames.com/health
```

---

## Output Files

Save documentation to:
- /root/migration-docs/docs/infrastructure/DEV-V26-SUBDOMAIN-SETUP.md

---

## Acceptance Criteria

- [ ] All 3 subdomains accessible via HTTPS
- [ ] SSL certificates auto-provisioned by Caddy
- [ ] CORS working between dev-app and dev-api
- [ ] Health endpoint responding on dev-api
- [ ] Documentation created

---

## Notes

- DO NOT modify production subdomains
- DO NOT modify staging subdomains
- Keep existing configuration intact
- DNS must be configured before testing

