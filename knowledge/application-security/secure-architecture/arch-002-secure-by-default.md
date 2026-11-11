---
id: ARCH-002
title: Secure by default configuration
type: principle
strength: strongly-recommended
evidence: established
category: secure-architecture
summary: Ship and deploy with the safest settings enabled out of the box. Require an explicit, deliberate action to loosen security, avoid default credentials, and leave unneeded features, ports and endpoints off.
lifecycle: [design, implementation, deployment, review]
standards: [CWE-1188, OWASP-Top10-2021:A05, NIST-SP-800-218]
tags: [secure-defaults, configuration, hardening, attack-surface, default-credentials]
contexts:
  - Designing configuration options for a service or library
  - Preparing a production deployment
  - Reviewing sample or development settings that might reach production
relationships:
  - type: supports
    target: ARCH-001
    note: Safe defaults mean a missed configuration step leaves a protected system, not an open one.
  - type: related_to
    target: LOG-005
    note: Verbose error output must be off unless explicitly enabled.
  - type: related_to
    target: API-007
    note: Unneeded and forgotten endpoints and versions enlarge the attack surface.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Secure_Product_Design_Cheat_Sheet.html
  - https://owasp.org/Top10/A05_2021-Security_Misconfiguration/
  - https://cwe.mitre.org/data/definitions/1188.html
last_reviewed: "2026-11-08"
---

Make the default state the secure state. A system that is safe only after someone remembers to harden it will, sooner or later, be deployed unhardened.

## How to apply

- **Safe defaults, explicit opt-out.** Authentication on, deny by default ([[AUTHZ-001]]), TLS on ([[CRYPTO-004]]), restrictive cookie settings ([[SESS-004]]), debug and verbose errors off ([[LOG-005]]). Loosening a setting should need a named, reviewable change, not the absence of one.
- **No default credentials.** Do not ship a built-in account or key with a shared value. Force the operator to set a unique secret at setup or generate one per installation ([[CRYPTO-003]]).
- **Minimal surface.** Enable only the features, modules, ports, protocols, admin interfaces and sample content that the deployment needs. Disable or remove the rest; unused code cannot be attacked if it is not running.
- **One production profile.** Keep development conveniences (debug toggles, permissive CORS, test accounts, open admin routes) out of the production configuration, not merely switched off by an environment variable that may be missing.
- **Safe when unset.** If a setting is absent or unparseable, fall back to the restrictive value, or refuse to start ([[ARCH-004]]).
- **Check it.** Verify security-relevant settings automatically at deploy time and in tests, so drift from the secure baseline is caught.

```
config.debug         = false          # must be set true explicitly, never in prod
config.admin_ui      = disabled
config.allowed_origins = []           # empty means none, not "any"
```

**Applies when** you author configuration schemas, templates or infrastructure definitions, as well as when you deploy them. Usability matters, but friction on the insecure path is the point.
