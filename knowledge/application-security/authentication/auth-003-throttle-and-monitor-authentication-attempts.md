---
id: AUTH-003
title: Throttle and monitor authentication attempts
type: pattern
strength: strongly-recommended
evidence: established
category: authentication
summary: Limit failed login, code-entry and recovery attempts per account and per source, slow attackers with backoff rather than permanent lockout, and log and alert on abnormal failure patterns.
lifecycle: [design, implementation, deployment, maintenance]
standards: [CWE-307, OWASP-ASVS:V6, NIST-SP-800-63B]
tags: [rate-limiting, brute-force, lockout, backoff, captcha, login]
contexts:
  - Login, MFA code entry and password reset endpoints
  - Any credential or one-time-code verification step
relationships:
  - type: mitigates
    target: AUTH-004
    note: Per-account and per-source limits raise the cost of guessing and spraying.
  - type: depends_on
    target: LOG-001
    note: Monitoring requires authentication events to be logged with context.
  - type: related_to
    target: API-002
  - type: related_to
    target: LOG-007
  - type: related_to
    target: AUTH-006
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
  - https://cheatsheetseries.owasp.org/cheatsheets/Credential_Stuffing_Prevention_Cheat_Sheet.html
  - https://cwe.mitre.org/data/definitions/307.html
  - https://pages.nist.gov/800-63-3/sp800-63b.html
last_reviewed: "2026-11-07"
---

Put a limit on how many credential or one-time-code guesses an attacker can make, and watch the failures. Apply this to every endpoint that checks a secret: login, MFA code entry, password reset tokens, API key checks.

**Why:** without limits, online guessing is only bounded by network speed. Naive lockouts, however, let an attacker deliberately lock out real users (a denial of service), so the design needs care.

**Do this**

- Count failures per account **and** per source (IP, device fingerprint, network range) and per credential-pair velocity across the system. Per-account counters alone miss password spraying; per-IP counters alone miss distributed attacks ([[AUTH-004]]).
- Prefer progressive delay, temporary lockout with automatic expiry, or step-up challenges over indefinite hard lockout. Keep the response for a throttled known account indistinguishable from an unknown one ([[AUTH-006]]).
- Keep limits on code-entry steps tight (one-time codes have small search spaces).
- Use bot-mitigation challenges as an extra signal after repeated failures, not as the only control.
- Log failures, lockouts and successes after many failures with account, source and timestamp, never the submitted password ([[LOG-001]], [[LOG-002]]). Alert on spikes in failures, many accounts hit from one source, or a success following a long failure run ([[LOG-007]]).
- Notify users of suspicious successful logins and of repeated failed attempts where it will not leak information.

```
on_failed_login(account, source):
    incr(fail[account]); incr(fail[source])
    delay = backoff(fail[account])           # grows, capped, expires
    if fail[source] > SOURCE_LIMIT: challenge_or_block(source)
    respond(generic_failure)                 # same shape regardless of account
```

**Applies when** any attacker can reach the verification endpoint, i.e. nearly always for internet-facing systems.
