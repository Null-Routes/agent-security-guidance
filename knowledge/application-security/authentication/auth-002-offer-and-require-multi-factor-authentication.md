---
id: AUTH-002
title: Offer and require multi-factor authentication for sensitive access
type: pattern
strength: strongly-recommended
evidence: established
category: authentication
summary: Support a second authentication factor for all accounts, require it for administrative and high-risk access, and prefer phishing-resistant factors over SMS or email codes.
lifecycle: [architecture, design, implementation]
standards: [CWE-308, OWASP-ASVS:V6, NIST-SP-800-63B]
tags: [mfa, 2fa, totp, webauthn, passkeys, phishing-resistant, step-up]
contexts:
  - Administrative interfaces and privileged roles
  - Accounts holding money, personal data or other sensitive data
  - Step-up before high-risk actions
relationships:
  - type: mitigates
    target: AUTH-004
    note: A stolen or guessed password alone no longer grants access.
  - type: related_to
    target: AUTH-005
    note: Recovery flows must not become a path around the second factor.
  - type: related_to
    target: AUTH-008
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html
  - https://pages.nist.gov/800-63-3/sp800-63b.html
  - https://cwe.mitre.org/data/definitions/308.html
last_reviewed: "2026-11-07"
---

Make a second factor available to every account, require it for privileged roles and sensitive operations, and choose the strongest factor your users can realistically use.

**Why:** passwords are routinely phished, reused and leaked. A second, independent factor turns a stolen password from full account takeover into a partial failure.

**Do this**

- Require MFA for administrators, operators and anyone with access to bulk or sensitive data. For ordinary users, offer enrollment prominently and consider requiring it based on risk or data sensitivity.
- Prefer phishing-resistant authenticators (public-key credentials such as WebAuthn/passkeys, hardware keys). Authenticator-app one-time codes are a reasonable middle option. SMS and email codes are weaker (SIM swap, mailbox compromise); if offered, treat them as a fallback and not for the highest-risk accounts.
- Verify the second factor server-side and bind it to the same login attempt; do not let a client skip the step by calling a later endpoint directly ([[AUTHZ-005]]).
- Apply attempt limits to code entry ([[AUTH-003]]) and protect against push-notification fatigue (rate-limit prompts, require number matching where available).
- Re-prompt (step-up) before especially sensitive actions such as changing credentials, payout details or security settings.
- Issue single-use recovery codes at enrollment, and make factor changes and removals go through a recovery path that is no weaker than the login itself ([[AUTH-005]]). Notify the user when factors change.
- Start a fresh session after the factor succeeds ([[SESS-002]]).

**Applies when** accounts hold meaningful value or privilege. For very low-risk, anonymous-friendly services the cost to users may outweigh the benefit; document that decision.
