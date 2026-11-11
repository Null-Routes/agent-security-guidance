---
id: ARCH-008
title: Relying on security through obscurity
type: anti-pattern
strength: strongly-recommended
evidence: established
category: secure-architecture
summary: Hiding a URL, port, algorithm or implementation detail and treating that as the access control fails once the secret is found or guessed. Use real authentication, authorization and vetted cryptography; obscurity may be an extra layer only.
lifecycle: [design, implementation, review]
standards: [CWE-656]
tags: [obscurity, hidden-endpoints, secret-algorithm, kerckhoffs, access-control]
contexts:
  - Admin or internal endpoints protected only by an unlisted path
  - Custom or proprietary "encoding", cipher or token scheme
  - Controls that depend on attackers not knowing how the system works
relationships:
  - type: violates
    target: ARCH-001
    note: One concealed detail is a single weak layer with no independent control behind it.
  - type: violates
    target: AUTHZ-001
    note: An unlisted endpoint without an access rule is effectively open.
  - type: violates
    target: CRYPTO-001
    note: A secret, homemade algorithm replaces public, reviewed cryptography.
sources:
  - https://cwe.mitre.org/data/definitions/656.html
  - https://cheatsheetseries.owasp.org/cheatsheets/Secure_Product_Design_Cheat_Sheet.html
last_reviewed: "2026-11-08"
---

Do not let secrecy of design or location be the thing that protects an asset. Assume an attacker knows how your system works, including its URLs, protocols, algorithms and source code. Protection must come from enforced controls and from keys that are real secrets.

## Negative example

```
# BAD: "nobody will find this URL"
route "/internal-9f3a7c/export-all-users":   # no authentication or authorization
    return dump(users)

# BAD: secret home-grown "encryption" as the only protection
function protect(data):
    return base64(xor(data, "myCompanySecret"))   # algorithm and key both fixed in code
```

Why it fails: hidden paths leak through links, logs, browser history, referrer headers, JavaScript bundles, scanners and guessing. A secret algorithm is recovered by anyone with the binary or a few samples, and its strength was never reviewed. Once the detail is known, every deployment is exposed at once and there is nothing else to stop the attacker.

## Corrected approach

```
route "/admin/export-users":
    require_authenticated()
    require_permission(caller, "users:export")   # server-side, deny by default
    audit_log(caller, "export")
    return export(users)

ciphertext = aead_encrypt(key_from_secret_store, data)   # vetted library
```

- Enforce authentication and authorization on every endpoint regardless of whether it is advertised ([[AUTHZ-001]], [[AUTHZ-003]]).
- Use public, reviewed cryptography with a properly managed secret key ([[CRYPTO-001]], [[CRYPTO-006]]).
- Design as if the architecture diagram and source were public.

## What is acceptable

Keys, passwords and tokens are legitimate secrets: their secrecy is the control, and it must be managed properly ([[CRYPTO-002]]). Obscurity is also fine as an additional layer that costs little: generic error messages ([[LOG-004]]), non-default ports, removing version banners, opaque identifiers. These raise attacker effort and reduce noise, but only on top of real controls ([[ARCH-001]]), and you must be willing to lose them without loss of security.
