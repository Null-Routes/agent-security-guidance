---
id: SESS-003
title: Invalidate sessions and tokens on logout, credential change and recovery
type: pattern
strength: strongly-recommended
evidence: established
category: session-management
summary: Terminate sessions on the server at logout, password or credential change, account recovery and detected compromise, and revoke related refresh tokens. Deleting the client cookie alone does not end a session.
lifecycle: [design, implementation, review, testing]
standards: [CWE-613, OWASP-ASVS:V7]
tags: [logout, revocation, session-termination, password-change, refresh-token]
contexts:
  - Implementing logout, password change and reset flows
  - Responding to suspected account compromise
  - Applications that issue long-lived refresh or remember-me tokens
relationships:
  - type: related_to
    target: AUTH-005
    note: A successful reset must revoke other sessions.
  - type: related_to
    target: SESS-008
    note: Self-contained tokens need a revocation or session check to be invalidated.
  - type: related_to
    target: SESS-005
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html
  - https://cwe.mitre.org/data/definitions/613.html
last_reviewed: "2026-11-08"
---

Make invalidation a server-side action: delete or mark the session record as ended, so that a copy of the identifier held by an attacker stops working. Clearing the cookie in the browser only removes the user's copy.

**Why:** if the credential stays valid on the server, "logout" gives a false sense of safety. A stolen cookie, a token on a lost device, or a session opened by an attacker with a leaked password continues to work after the victim signs out or changes their password.

## Invalidate when

- **Logout:** end the current session server-side and clear the cookie. Offer "sign out of all devices".
- **Password or credential change, MFA reset:** end all other sessions, keeping at most the current one, which should get a fresh identifier ([[SESS-002]]).
- **Account recovery or reset:** end all sessions and revoke refresh tokens and remember-me tokens ([[AUTH-005]]).
- **Privilege or role removal, account disablement, deletion:** end affected sessions immediately.
- **Suspected compromise or user report:** provide a way to list and revoke active sessions (device, location, last seen).
- **Timeouts:** treat expiry as invalidation ([[SESS-005]]).

## What to do

- Keep a server-side session record, or a revocation list or per-user "tokens valid after" timestamp, so invalidation takes effect before a token's natural expiry ([[SESS-008]]).
- Revoke refresh tokens and API tokens issued to the session; for refresh-token rotation, revoke the whole token family if a reused token is detected.
- Check the revocation state on each request, or at least within a short, known window.
- Log invalidation events ([[LOG-001]]) and notify the user of credential changes.
- Test: log out, then replay the old cookie or token and confirm it is rejected.

```text
on_password_change(user, current_session):
  sessions.delete_where(user = user AND id != current_session)
  tokens.revoke_all(user)
```
