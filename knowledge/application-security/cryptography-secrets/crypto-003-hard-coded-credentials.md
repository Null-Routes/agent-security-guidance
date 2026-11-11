---
id: CRYPTO-003
title: Hard-coded credentials and keys
type: anti-pattern
strength: mandatory
evidence: established
category: cryptography-secrets
summary: Embedding passwords, API keys or cryptographic keys in source, config defaults or images exposes them to everyone with access to the artifact. Load them from a secret store at runtime and rotate any that leaked.
lifecycle: [implementation, review, deployment, maintenance]
standards: [CWE-798, CWE-321, OWASP-ASVS:V11, OWASP-Top10-2021:A02]
tags: [hard-coded, credentials, api-keys, secret-scanning, version-control]
contexts:
  - Code that authenticates to a database, API or cloud service
  - Encryption or signing code with a key constant
  - Default or "temporary" credentials shipped in products and examples
relationships:
  - type: violates
    target: CRYPTO-002
    note: The managed-store pattern exists to avoid this practice.
  - type: violates
    target: CRYPTO-006
    note: A key compiled into code cannot be rotated or revoked on its own.
  - type: related_to
    target: AUTH-001
    note: Hard-coded password checks bypass proper credential storage.
sources:
  - https://cwe.mitre.org/data/definitions/798.html
  - https://cwe.mitre.org/data/definitions/321.html
  - https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html
last_reviewed: "2026-11-07"
---

Do not write credentials or cryptographic keys into code, committed configuration, default settings, images or client-side bundles. Supply them at runtime from a managed store ([[CRYPTO-002]]). This is mandatory because the exposure is near-universal and the cost of avoiding it is low: anyone who can read the artifact (repository, image layer, binary, shipped app, log of a build) obtains the secret, and it cannot be changed without a redeploy.

## Negative example

```
# BAD: secret and key live in the source
DB_PASSWORD = "EXAMPLE-NOT-A-REAL-PASSWORD"
ENCRYPTION_KEY = "EXAMPLE-NOT-A-REAL-KEY"

function connect():
    return db.open(host, "admin", DB_PASSWORD)

function is_admin(user, pass):
    return pass == "EXAMPLE-BACKDOOR-PASSWORD"   # same value on every installation
```

Problems: the value is visible in version control history forever, identical across every deployment, shared by everyone who can read the code, and impossible to rotate individually. Binaries are not safe either, since strings and constants can be extracted.

## Corrected approach

```
function connect():
    password = secret_store.get("db/app-password")   # identity-scoped, audited
    return db.open(host, service_account, password)
```

- Fetch per-environment, per-service credentials at runtime; use unique values for each deployment. Never ship a shared default or backdoor credential; require setup to create one.
- Add secret scanning to pre-commit and CI.
- If a secret was ever committed, treat it as compromised: revoke and rotate it first, then remove it from files and history. Rewriting history alone is not sufficient, because clones and caches may still hold it.
- Use placeholders like the ones above in tests and docs, never real values.

Related: [[CRYPTO-006]] covers rotating keys, and [[LOG-002]] covers keeping secrets out of logs.
