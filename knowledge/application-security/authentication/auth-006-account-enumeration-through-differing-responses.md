---
id: AUTH-006
title: Account enumeration through differing responses
type: anti-pattern
strength: recommended
evidence: established
category: authentication
summary: Telling clients whether a username exists, through different messages, status codes or timing in login, registration and recovery, lets attackers build target lists. Return uniform responses and equalize work.
lifecycle: [design, implementation, testing, review]
standards: [CWE-204, OWASP-ASVS:V6]
tags: [enumeration, username, information-disclosure, timing, login, registration]
contexts:
  - Login, password reset and registration endpoints
  - Any endpoint that accepts an identifier before authentication
relationships:
  - type: violates
    target: LOG-004
    note: Specific failure reasons in client responses should stay server-side.
  - type: related_to
    target: AUTH-004
    note: Enumeration feeds credential stuffing and spraying target lists.
  - type: related_to
    target: AUTH-005
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
  - https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html
  - https://cwe.mitre.org/data/definitions/204.html
last_reviewed: "2026-11-07"
---

Do not let an unauthenticated client distinguish "this account exists" from "this account does not exist". Different messages, HTTP statuses, response sizes or response times all count.

**Negative example**

```
POST /login
  unknown user  -> 404 {"error": "No such user"}
  wrong password -> 401 {"error": "Incorrect password"}

POST /forgot-password
  unknown email -> "We have no account with that address"

POST /register
  taken name    -> "That username is already registered"

# and: unknown user returns instantly, known user pays for a password hash (timing leak)
```

**Why it fails:** an attacker can test large lists of names or emails and learn which are real, then target those with stuffing or spraying ([[AUTH-004]]), phishing, or social engineering. Existence of an account can also be sensitive on its own (for example, membership of a support or medical service).

**Corrected approach**

```
POST /login           -> 401 {"error": "Invalid username or password"}   # same for both cases
POST /forgot-password -> 200 "If the address is registered, instructions were sent"
login():  always run password_verify against a dummy hash when the user is missing
```

- Use one message and one status for all authentication failures, including locked or throttled accounts ([[AUTH-003]]).
- Equalize work: verify against a dummy hash when the user does not exist; send recovery email asynchronously so latency does not vary.
- For registration, accept the request and email the address owner ("complete sign-up" or "you already have an account"), or require email verification before revealing anything. If a unique-username error is unavoidable, rate-limit it and accept the residual risk consciously.
- Return distinguishable details only to authenticated or verified parties, and log the real reason server-side ([[LOG-004]]).

**Applies when** account existence is not already public. If usernames are intentionally public (a social profile URL), focus on rate limits instead.
