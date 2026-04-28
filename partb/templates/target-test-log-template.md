# Target Test Log

> Copy this template when creating a detailed folder under `partb/targets/<target-name>/`.

---

## 1. Program Intake & Scope Lock

| Field | Detail |
|---|---|
| **Platform** | e.g. HackerOne / Bugcrowd / Immunefi / Intigriti / YesWeHack |
| **Program name** | |
| **HackerOne program URL** | |
| **Program bounty URL** | |
| **Official website URL** | |
| **Scope page URL** | |
| **Welcomed vulnerability types** | |
| **Rules & restrictions** | e.g. use only your own account(s), no automated scanning |
| **Date started** | YYYY-MM-DD |
| **Team member(s)** | |

### Scope Boundaries

| Allowed (In Scope) | Prohibited (Out of Scope) |
|---|---|
| | |

---

## 2. Attack Surface Mapping

Map reachable surfaces before deep testing:

| Surface | URL / Endpoint | Notes |
|---|---|---|
| Homepage | | |
| Login | | |
| Signup | | |
| Password Reset | | |
| Invitation Flows | | |
| Onboarding | | |
| Reports / Dashboard | | |
| App / Developer Areas | | |
| Billing / Subscription | | |
| Sharing / Collaboration | | |
| API / Docs | | |
| Other | | |

---

## 3. Pre-auth UI Review

Review public pages/flows for obvious unauthenticated issues:

| Page / Flow | Observation | Suspicious Parameters / Objects | Candidate? |
|---|---|---|---|
| | | | Y/N |

---

## 4. Pre-auth DevTools Deep Dive

Inspect rendered DOM, XHR/fetch traffic, tokens, nonces, state values, hidden params, exposed endpoints.

| Component | Findings |
|---|---|
| **DOM inspection** (hidden fields, embedded state, exposed IDs) | |
| **XHR/fetch traffic** (object IDs, tokens, nonces, `state` values) | |
| **Hidden parameters** | |
| **Embedded state objects** | |
| **Exposed routes / endpoints** | |
| **Token / nonce quality** | |

---

## 5. Baseline Common-Vulnerability Pack

### 5.1 Access Control / IDOR / Object Exposure

Quick checks:
- [ ] Change UUID/ID in URL — does it return another user's data?
- [ ] Access authenticated pages while unauthenticated (direct URL)
- [ ] Change `user_id`, `account_id`, `workspace_id` params in requests
- [ ] API enumeration: `/api/users/<id>`, `/api/accounts/<id>`

Details:

| Test | URL / Request | Result |
|---|---|---|
| | | |

### 5.2 Authentication / Session / Invite / Reset / OAuth

Quick checks:
- [ ] Password reset token predictable? (timestamp, short hex, sequential)
- [ ] OAuth `redirect_uri` — can it be modified to an attacker domain?
- [ ] JWT `alg=none` bypass attempt
- [ ] Invitation token reusable? Can invite be accepted by a different email?
- [ ] Login response leaks account existence (timing or message diff)

Details:

| Test | URL / Request | Result |
|---|---|---|
| | | |

### 5.3 Token / Nonce / Randomness / Binding Quality

Quick checks:
- [ ] CSRF token present on state-changing requests? Can it be omitted?
- [ ] JWT signature actually verified? Try modifying payload with same signature
- [ ] OAuth `state`/`nonce` reused across requests? Predictable?
- [ ] Password reset / invite links use secure random tokens?

Details:

| Test | URL / Request | Result |
|---|---|---|
| | | |

### 5.4 Input Handling / XSS / Unsafe Reflection

Quick checks:
- [ ] Reflected parameters echo `< > " '` back in page source unescaped?
- [ ] File upload name rendered in page? Try `<svg onload=alert(1)>` in filename
- [ ] Search/query params reflected in page with HTML context?
- [ ] `Origin`/`Referer` headers reflected in CORS headers unsanitized?
- [ ] JSONP endpoints? URL hash reflection?

Details:

| Test | URL / Request | Result |
|---|---|---|
| | | |

### 5.5 Misconfiguration / Unsafe Exposure / Business Logic

Quick checks:
- [ ] API responses leak internal IDs, emails, or sensitive fields?
- [ ] Admin/api/internal paths accessible without auth? Try `/admin`, `/api/internal`, `/graphql`
- [ ] CORS: `Access-Control-Allow-Origin: *` with credentials?
- [ ] Price/coupon/amount parameter modifiable in requests?
- [ ] Rate limiting absent on login, reset, invite endpoints?
- [ ] Verbose error messages leaking stack traces, DB errors, internal paths?

Details:

| Test | URL / Request | Result |
|---|---|---|

---

## 6. Login Decision

| Question | Answer |
|---|---|
| Do pre-auth and DevTools rounds suggest authenticated testing is valuable? | Y/N |
| Account type needed | e.g. free trial, registered user |
| Login method | e.g. email/password, Google OAuth, SSO |

---

## 7. Post-auth State Confirmation

| Item | Observation |
|---|---|
| Account state (role, tier, trial status) | |
| Newly unlocked navigation/features | |
| Project/workspace/report/app surfaces | |
| Subscription / billing gate | |

---

## 8. Post-auth Core Checks

Re-run baseline pack (5.1–5.5) against authenticated features.

| Category | Test | Result |
|---|---|---|
| Access Control / IDOR | | |
| Auth / Session | | |
| Token / Nonce | | |
| XSS / Input | | |
| Misconfiguration / Business Logic | | |

---

## 9. Multi-account / Multi-role Decision

| Question | Answer |
|---|---|
| Does the target have collaboration/invitation/sharing/ownership boundaries? | Y/N |
| Worth testing cross-account access? | Y/N |
| Second account requested? | Y/N |

---

## 10. Evidence & Submission Readiness

Before calling something a candidate finding:

- [ ] Reproducible in clean session / incognito
- [ ] Scope proof captured (program page, scope page screenshots)
- [ ] Impact explained: who is affected, what is compromised, likely consequence
- [ ] Screenshots/request logs saved in this folder
- [ ] Evidence sufficient to defend in presentation and Q&A

---

## 11. Finding Summary

| Field | Detail |
|---|---|
| **Target** | |
| **Vulnerability** | |
| **Root Cause** | |
| **Impact** | |
| **Finding Type** | Type 1 (confirmed non-zero-day) / Type 2 (zero-day candidate) |
| **Platform Severity** | |
| **Normalized Severity (S0–S4)** | |
| **CVSS v3.1** (fallback) | Vector: / Score: |
| **Mitigation Suggestion** | |

---

## Severity Justification

| Criterion | Assessment |
|---|---|
| **Severity score (0–6)** | S_ (__) |
| **Impact-evidence score (0–2)** | ___ — exploit path, affected asset/data, real-world consequence |
| **Novelty score (0–2)** | ___ — Type 1 or Type 2 with justification |

---

## Zero-day Novelty Check (for Type 2 claims)

| Search Source | Query Used | Result |
|---|---|---|
| Google | | e.g. no prior report found |
| HackerOne Hacktivity | | |
| CVE / NVD | | |
| Exploit-DB | | |
| GitHub Issues (target repo) | | |
| Target's own changelog / security advisories | | |

---

## AI Usage Log

> Record only when AI materially assisted in finding, validating, or documenting a candidate vulnerability.

| Date | Tool | Prompt/Use Summary | Role in Finding |
|---|---|---|---|
| | | | e.g. suggested IDOR test vector, validated JWT decode |
