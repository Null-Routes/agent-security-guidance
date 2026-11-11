---
id: AUTH-004
title: Credential stuffing and password spraying
type: vulnerability
strength: strongly-recommended
evidence: established
category: authentication
summary: Attackers replay breached username/password pairs or try a few common passwords across many accounts. Defend with MFA, breached-password screening, layered throttling and detection of automated login traffic.
lifecycle: [design, implementation, deployment, maintenance]
standards: [CWE-307, CWE-521, CAPEC-600, CAPEC-565, OWASP-Top10-2021:A07, OWASP-ASVS:V6, NIST-SP-800-63B]
tags: [credential-stuffing, password-spraying, account-takeover, breached-passwords, bots]
contexts:
  - Any internet-facing login
  - Systems where users choose their own passwords
relationships:
  - type: related_to
    target: AUTH-002
    note: MFA is the strongest single mitigation.
  - type: related_to
    target: AUTH-003
  - type: related_to
    target: AUTH-001
    note: Hashing does not stop online replay, but limits damage from offline cracking.
  - type: related_to
    target: LOG-007
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Credential_Stuffing_Prevention_Cheat_Sheet.html
  - https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
  - https://pages.nist.gov/800-63-3/sp800-63b.html
  - https://cwe.mitre.org/data/definitions/521.html
last_reviewed: "2026-11-07"
---

**Credential stuffing** replays username/password pairs leaked from other services, exploiting password reuse. **Password spraying** tries a small set of very common passwords against many accounts so that no single account trips a lockout. Both are automated, use distributed infrastructure, and succeed even when your own code is correct.

**How it shows up:** bursts of login attempts with high failure rates, many distinct usernames from few sources (or the reverse, one username from many sources), logins using lists of valid-looking email addresses, and unusual user agents or timing.

**Prevention**

- Offer and require MFA where risk warrants it ([[AUTH-002]]).
- Check new and changed passwords against known-breached and commonly used password lists, and enforce a sensible minimum length instead of arbitrary composition rules. Allow long passphrases and password managers (paste, all printable characters).
- Throttle by account, by source and across the whole system, and challenge suspicious traffic ([[AUTH-003]]).
- Keep failure responses uniform so attackers cannot cheaply learn which usernames exist ([[AUTH-006]]).
- Detect and alert on the patterns above, and watch for successful logins from new devices or locations ([[LOG-007]]).
- Make post-takeover damage smaller: require re-authentication for credential and payout changes, and notify users of sensitive account changes.
- Store passwords with a slow salted hash so a leak of your own database does not feed the next campaign ([[AUTH-001]]).

**Applies when** you authenticate with passwords. Passwordless or federated sign-in changes the attack surface but does not remove it from the identity provider ([[AUTH-008]]).

Do not rely on a single control such as CAPTCHA or IP blocking; attackers rotate addresses and solve challenges at scale.
