# Task 002: API Endpoint QA Testing

**Status:** Pending  
**Priority:** High  
**Assigned To:** Paperclip Agent (Claude)  
**Created:** 2026-04-16  
**Depends On:** Task 001 (Dev Subdomains)  

---

## Objective

Create comprehensive QA test suite and test all 60+ API endpoints on dev-v26 environment.

---

## Endpoints to Test

### Auth Endpoints

| Method | Endpoint | Auth | Test Priority |
|--------|----------|------|---------------|
| POST | /auth/check-user | No | High |
| POST | /auth/send-magic-link | No | High |
| POST | /auth/verify-magic-link | No | High |
| POST | /auth/logout | Yes | High |
| GET | /users/me | Yes | High |
| PATCH | /users/me | Yes | Medium |

### Game Endpoints

| Method | Endpoint | Auth | Test Priority |
|--------|----------|------|---------------|
| GET | /games | Yes | High |
| GET | /games/categories | Yes | High |
| GET | /games/category/:category | Yes | Medium |
| GET | /games/slug/:slug | Yes | Medium |
| GET | /games/:id | Yes | Medium |

### Order Endpoints

| Method | Endpoint | Auth | Test Priority |
|--------|----------|------|---------------|
| GET | /orders | Yes | High |
| POST | /orders | Yes | High |
| GET | /orders/:id | Yes | Medium |
| PATCH | /orders/:id | Yes | Medium |
| DELETE | /orders/:id | Yes | High |
| POST | /orders/:id/reorder | Yes | Medium |

### Credit Endpoints

| Method | Endpoint | Auth | Test Priority |
|--------|----------|------|---------------|
| GET | /credits/balance | Yes | High |
| GET | /credits/transactions | Yes | Medium |
| POST | /credits/purchase | Yes | High |
| GET | /api/settings/pricing | Yes | Medium |
| POST | /api/settings/calculate-price | Yes | Medium |

### Consultation Endpoints

| Method | Endpoint | Auth | Test Priority |
|--------|----------|------|---------------|
| GET | /api/consultations/slots | Yes | Medium |
| POST | /api/consultations/book | Yes | High |
| GET | /api/consultations/my | Yes | Medium |
| DELETE | /api/consultations/:id | Yes | Medium |

### Friends/Affiliate Endpoints

| Method | Endpoint | Auth | Test Priority |
|--------|----------|------|---------------|
| GET | /api/friends/status | Yes | Medium |
| POST | /api/friends/activate | Yes | High |
| GET | /api/friends/stats | Yes | Medium |
| GET | /api/friends/referrals | Yes | Medium |
| GET | /api/friends/earnings | Yes | Medium |
| POST | /api/friends/withdrawals | Yes | Medium |

### Admin Endpoints

| Method | Endpoint | Auth | Test Priority |
|--------|----------|------|---------------|
| GET | /admin/stats | Admin | High |
| GET | /admin/activity/feed | Admin | Medium |
| GET | /admin/users | Admin | High |
| PATCH | /admin/users/:id | Admin | Medium |
| GET | /admin/orders | Admin | High |
| PATCH | /admin/orders/:id | Admin | Medium |
| PATCH | /admin/consultations/:id | Admin | Medium |
| GET | /admin/settings/pricing | Super | Medium |
| PATCH | /admin/settings/pricing | Super | High |
| GET | /admin/affiliates | Admin | Medium |
| GET | /admin/withdrawals | Admin | Medium |
| PATCH | /admin/withdrawals/:id | Admin | Medium |

---

## Test Environment

```
Base URL: https://dev-api-v26.eventxgames.com
Frontend: https://dev-app-v26.eventxgames.com
```

---

## Test Steps

### Step 1: Health Check

```bash
curl -s https://dev-api-v26.eventxgames.com/health | jq
```

Expected: `{"status": "ok"}`

### Step 2: Public Endpoints (No Auth)

```bash
# Check user existence
curl -X POST https://dev-api-v26.eventxgames.com/auth/check-user \
  -H "Content-Type: application/json" \
  -d {email: test@example.com}
```

### Step 3: Authenticated Endpoints

```bash
# Get auth token first (manual step or use test account)
TOKEN="<firebase-token>"

# Test games endpoint
curl -s https://dev-api-v26.eventxgames.com/games \
  -H "Authorization: Bearer $TOKEN" | jq
```

### Step 4: Admin Endpoints

```bash
# Use admin token
ADMIN_TOKEN="<admin-firebase-token>"

curl -s https://dev-api-v26.eventxgames.com/admin/stats \
  -H "Authorization: Bearer $ADMIN_TOKEN" | jq
```

---

## Output Files

Create these files in /root/migration-docs/docs/qa/:

1. **API-TEST-SUITE.md** - Complete test commands
2. **QA-CHECKLIST.md** - Pass/fail checklist
3. **QA-REPORT-2026-04-16.md** - Test results

---

## QA Checklist Template

| # | Endpoint | Method | Status | Response Time | Notes |
|---|----------|--------|--------|---------------|-------|
| 1 | /health | GET | | | |
| 2 | /auth/check-user | POST | | | |
| 3 | /games | GET | | | |
| ... | ... | ... | | | |

---

## Acceptance Criteria

- [ ] All public endpoints tested
- [ ] Sample authenticated endpoints tested
- [ ] Response times documented
- [ ] Errors documented
- [ ] QA report created
- [ ] Checklist updated

---

## Notes

- Use dev-v26 environment only
- Do not test on production
- Document any failures
- Report blocking issues immediately

