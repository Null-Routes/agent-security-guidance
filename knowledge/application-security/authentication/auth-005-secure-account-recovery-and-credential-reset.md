---
id: AUTH-005
title: Secure account recovery and credential reset
type: pattern
strength: strongly-recommended
evidence: established
category: authentication
summary: Make recovery no weaker than login. Send single-use, short-lived, random reset tokens to a verified channel, answer uniformly, avoid knowledge-based questions, and revoke sessions and notify the user after a reset.
lifecycle: [design, implementation, review]
standards: [CWE-640, OWASP-ASVS:V6, NIST-SP-800-63B]
tags: [password-reset, account-recovery, reset-token, email, takeover]
contexts:
  - Forgotten password and lost-factor flows
  - Email or phone number change flows
relationships:
  - type: depends_on
    target: CRYPTO-007
    note: Reset tokens must come from a secure random source.
  - type: related_to
    target: AUTH-006
    note: Recovery forms are a common enumeration oracle.
  - type: related_to
    target: AUTH-002
  - type: related_to
    target: SESS-003
  - type: related_to
    target: DATA-006
    note: Tokens in URLs can leak through logs and referrers; limit exposure.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html
  - https://cheatsheetseries.owasp.org/cheatsheets/Choosing_and_Using_Security_Questions_Cheat_Sheet.html
  - https://cwe.mitre.org/data/definitions/640.html
last_reviewed: "2026-11-07"
---

Treat recovery as an alternate login path: whatever it accepts as proof of identity is what an attacker will target. Design it so it is not easier than normal authentication.

**Do this**

- Respond identically whether or not the account exists ("If that address is registered, we sent instructions"), with similar timing ([[AUTH-006]]).
- Generate a long, random, single-use reset token with a secure random source ([[CRYPTO-007]]). Store only a hash of it server-side, bind it to the account, and expire it within a short window. Invalidate it on use and when a newer one is issued.
- Deliver it only to the contact channel already on file, never to an address supplied in the reset request. Do not put secrets in a URL longer than necessary: use a single-purpose, short-lived link, and avoid third-party resources on the reset page so the token cannot leak via referrer headers ([[DATA-006]]).
- Do not use knowledge-based security questions; answers are guessable or discoverable.
- Rate-limit requests and token attempts ([[AUTH-003]]).
- Do not log users in automatically by token alone if MFA is enabled; require the second factor, or make sure lost-factor recovery has an equivalent identity-proofing step ([[AUTH-002]]).
- After a successful reset: apply the new password's normal quality checks, revoke other sessions and tokens ([[SESS-003]]), and notify the user through all known channels so a hijack is noticed.
- Never reveal or email the existing password; this should be impossible if passwords are hashed ([[AUTH-001]]).

**Applies when** users can regain access without an administrator. For high-assurance systems, recovery may legitimately require out-of-band identity proofing or an administrator.
