---
id: SESS-002
title: Issue a new session identifier at authentication and privilege change
type: pattern
strength: strongly-recommended
evidence: established
category: session-management
summary: Create a fresh session identifier and discard the old one whenever a user signs in or their privilege level changes, so an identifier planted or observed earlier cannot be used afterwards.
lifecycle: [design, implementation, review, testing]
standards: [CWE-384, CAPEC-61, OWASP-ASVS:V7]
tags: [session-regeneration, login, privilege-change, fixation, session-id]
contexts:
  - Login, step-up authentication and role or privilege changes
  - Moving from an anonymous session to an authenticated one
relationships:
  - type: mitigates
    target: SESS-006
  - type: related_to
    target: AUTH-002
    note: Start the fresh session after the factor completes.
  - type: related_to
    target: SESS-003
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html
  - https://cwe.mitre.org/data/definitions/384.html
last_reviewed: "2026-11-08"
---

When a session's authentication state or privilege changes, generate a new identifier ([[SESS-001]]), copy over only the data that should survive, invalidate the old identifier on the server, and send the new one to the client.

**Why:** anything that learned the old identifier (a fixation attack, a shared device, a network observer on an earlier insecure request, a leaked log line) would otherwise inherit the authenticated session. A new identifier at the moment of privilege gain cuts that link ([[SESS-006]]).

## When to regenerate

- After successful authentication, including after the final step of multi-factor sign-in ([[AUTH-002]]).
- On privilege elevation: step-up authentication, switching to an administrative role, or assuming another identity.
- On privilege reduction and sign-out, where practical ([[SESS-003]]).
- After password or credential changes and account recovery.

## What to do

- Always invalidate the previous identifier server-side. Merely setting a new cookie while leaving the old session valid defeats the purpose.
- Do not carry over unneeded pre-login state; if you keep a pre-login item (a shopping cart), copy it explicitly.
- Apply the same cookie attributes to the new identifier ([[SESS-004]]).
- Reset the idle and absolute timers for the new session ([[SESS-005]]).
- Test it: capture the identifier before login, authenticate, and confirm the old value no longer works and the new value differs.

```text
on_login_success(request, user):
  old = request.session_id
  new = new_session(user)         # fresh random identifier
  store.delete(old)
  respond.set_cookie(new)
```
