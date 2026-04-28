# Atolls — hotukdeals.com Test Log

> **Status:** 🟡 In Progress — Recon phase complete, awaiting human Browser/DevTools testing

---

## 1. Program Intake & Scope Lock

| Field | Detail |
|---|---|
| **Platform** | Intigriti |
| **Program name** | Atolls Vulnerability Disclosure Program (VDP) |
| **Program URL** | https://app.intigriti.com/programs/atolls/atollsvdp |
| **Official website URL** | https://www.hotukdeals.com |
| **Scope page URL** | https://app.intigriti.com/programs/atolls/atollsvdp/detail |
| **Welcomed vulnerability types** | All except explicitly out-of-scope |
| **Rules & restrictions** | Use @intigriti.me email for registration; max 2 req/sec; no automated scanners; stop on sensitive data discovery |
| **Date started** | 2026-04-28 |
| **Team member(s)** | Xing Yin |

### Scope Boundaries

| Allowed (In Scope) | Prohibited (Out of Scope) |
|---|---|
| hotukdeals.com (and 35+ other Atolls domains) | Self-XSS (unless exploitable to others) |
| Mobile apps owned by Atolls companies | Pre-Auth Account takeover/OAuth squatting |
| Self-registration with @intigriti.me email | CSRF with low impact |
| | Missing cookie flags / security headers |
| | Rate limiting absence |
| | Clickjacking without proven impact |
| | Username/email enumeration |
| | Blind SSRF without proven business impact |
| | DoS/DDoS |
| | Any domain not listed in scope |

---

## 2. Attack Surface Mapping

| Surface | URL / Endpoint | Notes |
|---|---|---|
| **Homepage** | https://www.hotukdeals.com | Deal listing, voting, comments |
| **Login** | https://www.hotukdeals.com/login | JS-rendered, not crawlable |
| **Register** | https://www.hotukdeals.com/register | JS-rendered |
| **Password Reset** | https://www.hotukdeals.com/forgot-password | Redirects to categories |
| **GraphQL** | https://www.hotukdeals.com/graphql | **LIVE** — no auth required for some queries |
| **REST API v2** | https://www.hotukdeals.com/rest_api/v2/ | **Fully documented publicly** — most endpoints require auth |
| **Admin** | https://www.hotukdeals.com/admin | Returns 404 |
| **Vouchers** | https://www.hotukdeals.com/vouchers/ | Voucher code system |
| **Voucher API** | https://www.hotukdeals.com/vouchers/api/ | Returns 404 |
| **Wishlist** | https://www.hotukdeals.com/wl/ | Wishlist feature |
| **Deal posting** | Post deal flow | Requires auth |
| **Commenting** | Thread comment system | Requires auth |
| **Q&A** | Weekly Q&A discussions | Interactive |
| **Alerts** | Deal alerts subscription | |
| **Discount for carers** | Third-party OAuth? | |

### Tech Stack
- **Platform:** "Pepper" (internal name)
- **API:** REST v2 at /rest_api/v2 (JSON, standard HTTP methods)
- **GraphQL:** Active at /graphql
- **Architecture:** SPA (React/Next.js likely), client-side rendered routes
- **CDN:** Cloudflare (robots.txt references /cdn-cgi/)

---

## 3. Pre-auth UI Review

> **Note:** Full UI review requires human browser. Key observations from HTML fetch:

| Page / Flow | Observation | Suspicious Parameters |
|---|---|---|
| Homepage | Threads with IDs, merchant IDs, user IDs in data attributes | `thread_id`, `merchant_id`, `user_id` — all integers |
| robots.txt | Exposed internal paths: `/admin`, `/graphql`, `/rest-api/`, `/bpn/getRequestPermissionParams` | Query params: `sortBy`, `priceFrom`, `priceTo`, `merchant-id` |
| REST API docs | Full API schema publicly accessible | Sequential IDs, user data in thread responses |

---

## 4. Pre-auth DevTools Deep Dive

> **Note:** AI agent limited to HTTP-level inspection. Human DevTools required for:
> - JS bundle analysis (GraphQL query extraction)
> - Network tab — capture actual GraphQL queries/mutations
> - DOM inspection — data attributes, embedded state, tokens

| Component | AI-Level Findings |
|---|---|
| **DOM inspection** | Not available via AI tooling |
| **XHR/fetch traffic** | Not available via AI tooling |
| **Hidden parameters** | Need DevTools |
| **Embedded state objects** | Need DevTools |
| **Exposed routes** | `/admin`, `/graphql`, `/rest_api/v2/`, `/vouchers/`, `/wl/` |
| **Token / nonce quality** | Need DevTools |

---

## 5. Baseline Common-Vulnerability Pack

### 5.1 Access Control / IDOR / Object Exposure

| Test | URL / Request | Result |
|---|---|---|
| GraphQL user query (user ID 1) | `GET /graphql?query=query {user(userId:1){username}}` | 🔴 **Returns data without auth:** `{"data":{"user":{"username":"Admin"}}}` |
| GraphQL user query (user IDs 2-1000) | Same query with different IDs | All return `{"data":{"user":null}}` |
| GraphQL user query — cross-brand | dealabs.com: `/graphql?query=query {user(userId:1){username}}` | 🔴 **Returns:** `{"data":{"user":{"username":"adealistrateur"}}}` |
| GraphQL user query — coupons.com | coupons.com/graphql | 404 — not a Pepper platform site |
| REST API thread list (no auth) | `GET /rest_api/v2/thread?limit=3` | 401 — requires authentication |
| REST API user (no auth) | `GET /rest_api/v2/user?user_id=1` | 401 — requires authentication |
| REST API group list (no auth) | `GET /rest_api/v2/group?list=popular` | 401 — requires authentication |

**Assessment:** Limited unauthenticated GraphQL access to user ID 1 only. Need post-auth testing to determine if full IDOR exists.

### 5.2 Authentication / Session / Invite / Reset / OAuth

| Quick Check | Status |
|---|---|
| Password reset token predictable? | Need human to test via browser |
| OAuth `redirect_uri` modifiable? | Need human to test via browser |
| JWT `alg=none` bypass | Need to capture tokens via DevTools |
| Invitation token reusable? | Need to test invite flow |
| Login response leaks account existence? | Need human to test via browser |

### 5.3 Token / Nonce / Randomness / Binding Quality

| Quick Check | Status |
|---|---|
| CSRF token present? | Need DevTools |
| JWT signature verified? | Need to capture JWT |
| OAuth state/nonce reused? | Need to capture OAuth flow |
| Secure random tokens? | Need DevTools |

### 5.4 Input Handling / XSS / Unsafe Reflection

| Quick Check | Status |
|---|---|
| Reflected parameters echo unescaped? | Need browser testing |
| File upload name rendered in page? | Need to test deal posting |
| Search/query reflection | Need to test search |
| CORS misconfiguration | Need DevTools |
| JSONP endpoints? | Need to check |

### 5.5 Misconfiguration / Unsafe Exposure / Business Logic

| Test | Result |
|---|---|
| API responses leak internal IDs? | REST API thread schema includes `user_id`, `thread_id`, `merchant_id` — sequential integers |
| Admin/api paths accessible? | `/admin` → 404; `/rest-api/` → public docs; `/graphql` → live |
| CORS `Access-Control-Allow-Origin: *`? | Need DevTools |
| Price/amount modifiable? | Deal system has `price`, `next_best_price`, `price_discount` — need post-auth |
| Rate limiting absent? | Rule says max 2 req/sec — suggests throttling exists |
| Verbose error messages? | ✅ GraphQL errors reveal field names (`"Did you mean 'images' or 'email'?"`, `"Field 'images' of type 'Pix!' must have a sub selection"`) |
| robots.txt exposure? | ✅ Exposes `/admin`, `/graphql`, `/rest-api/`, internal query params |
| REST API docs public? | ✅ Full API documentation accessible at `/rest_api/v2/` |

---

## 6. Login Decision

| Question | Answer |
|---|---|
| Do pre-auth rounds suggest authenticated testing is valuable? | **YES** — GraphQL endpoint live, REST API fully documented, sequential IDs suggest IDOR potential |
| Account type needed | Free registered user |
| Next action | **Register account on hotukdeals.com with @intigriti.me email** |

---

## Candidate Findings (Preliminary)

### CF-1: Unauthenticated GraphQL Admin User Data Exposure

| Field | Detail |
|---|---|
| **Target** | hotukdeals.com, dealabs.com (Pepper platform) |
| **Vulnerability** | GraphQL `user` query returns admin user data without authentication |
| **Evidence** | `GET /graphql?query=query {user(userId:1){username,createdAt}}` → `{"data":{"user":{"username":"Admin","createdAt":1092413700}}}` |
| **Impact** | Low — only user ID 1 accessible without auth (limited data: username + creation date). Not exploitable for user enumeration. |
| **Novelty** | Unknown — need to search for prior reports |
| **Severity (preliminary)** | S1 Low — limited impact, no sensitive data leaked |
| **Next step** | Re-test after authentication to determine if full IDOR exists |

---

## Next Actions for Human Tester

1. **Register account** on hotukdeals.com with `@intigriti.me` email
2. **DevTools — Network tab**: Capture all GraphQL queries/mutations, JWT tokens, session cookies
3. **DevTools — Sources tab**: Extract GraphQL schema from JS bundles
4. **Post-auth GraphQL testing**: Re-run user queries with auth cookies. Try user IDs beyond 1.
5. **IDOR on threads/comments**: Query thread IDs, check if you can access/edit other users' threads
6. **Voucher system**: Test coupon code claim/redemption for race conditions or reuse
7. **Cross-brand testing**: Same GraphQL test on pepper.com, chollometro.com, dealabs.com
8. **Password reset flow**: Check token predictability
9. **OAuth flow**: Test redirect_uri validation if OAuth login is available

---

## AI Usage Log

| Date | Tool | Prompt/Use Summary | Role in Finding |
|---|---|---|---|
| 2026-04-28 | opencode (DeepSeek) | Reconnaissance: robots.txt analysis, GraphQL endpoint discovery, REST API documentation review, cross-brand testing | Identified GraphQL endpoint, discovered unauthenticated user query, mapped attack surface |
