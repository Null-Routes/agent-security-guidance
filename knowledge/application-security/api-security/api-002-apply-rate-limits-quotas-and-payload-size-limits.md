---
id: API-002
title: Apply rate limits, quotas and payload size limits
type: pattern
strength: strongly-recommended
evidence: established
category: api-security
summary: Bound request rate, request size, pagination, query complexity and per-client quotas so one caller cannot exhaust CPU, memory, storage or paid downstream services.
lifecycle: [design, implementation, deployment]
standards: [OWASP-API-2023:API4, CWE-770, CWE-400]
tags: [api, rate-limiting, quotas, denial-of-service, resource-consumption]
contexts:
  - Public or partner-facing APIs
  - Endpoints that send email or SMS, call paid services, or run expensive queries
  - Endpoints that accept uploads, bulk operations or user-defined query shapes
relationships:
  - type: related_to
    target: AUTH-003
    note: Login endpoints need stricter, identity-aware throttling than the general limit.
  - type: related_to
    target: INPUT-010
    note: Upload size limits are one case of payload limits.
  - type: related_to
    target: LOG-007
    note: Limit hits are a useful signal to alert on.
  - type: supports
    target: ARCH-001
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Denial_of_Service_Cheat_Sheet.html
  - https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html
last_reviewed: "2026-11-07"
---

Put explicit ceilings on everything a client can make the server consume, and enforce them before expensive work starts. Return a clear throttling response (for example HTTP 429 with a retry hint) instead of silently degrading.

Unbounded consumption turns a normal endpoint into a denial-of-service or cost-amplification tool, and also makes brute-force and scraping practical.

## Limits to define

- **Request rate** per credential, per IP and per tenant, with tighter limits on sensitive or costly operations (password reset, OTP send, search, export).
- **Payload size**: maximum body size, header size, JSON nesting depth, array length and multipart parts. Reject early, ideally at the edge, before buffering the body.
- **Result size**: a default and maximum page size; never allow "return everything".
- **Query complexity**: depth and cost limits for GraphQL-style or user-composed queries; timeouts on database and downstream calls.
- **Quotas** for resources that cost money or storage: messages sent, files stored, batch item counts.
- **Concurrency**: cap parallel in-flight requests or jobs per caller.

```text
if size(request.body) > MAX_BODY:      return 413
if limiter.exceeded(caller, route):    return 429 (Retry-After)
if request.page_size > MAX_PAGE:       clamp or reject
```

## Notes

Key limits on the authenticated identity where possible; IP-only limits are easy to evade and can punish shared networks. Pick values from observed legitimate use plus headroom, and review them as usage changes. Rate limiting reduces abuse but does not replace authentication ([[AUTH-003]]) or authorization.
