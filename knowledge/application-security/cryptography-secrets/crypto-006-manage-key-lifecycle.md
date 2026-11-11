---
id: CRYPTO-006
title: "Manage the cryptographic key lifecycle: separation, rotation, revocation and destruction"
type: pattern
strength: strongly-recommended
evidence: established
category: cryptography-secrets
summary: Generate keys securely, use a separate key per purpose, store them apart from the data they protect, rotate on a schedule and after suspected compromise, and have tested procedures to revoke and destroy keys.
lifecycle: [architecture, design, deployment, maintenance]
standards: [CWE-321, NIST-SP-800-57, OWASP-ASVS:V11]
tags: [key-management, rotation, envelope-encryption, revocation, key-separation, kms]
contexts:
  - Designing encryption, signing or token systems with long-lived keys
  - Choosing where keys live and who may use them
  - Planning for compromise or algorithm migration
relationships:
  - type: related_to
    target: CRYPTO-002
    note: The secret store holds keys and credentials; this entry covers how keys are generated, rotated and retired.
  - type: related_to
    target: CRYPTO-005
    note: Authenticated encryption depends on sound key handling.
  - type: related_to
    target: CRYPTO-007
    note: Keys must be generated from a secure random source.
  - type: related_to
    target: AUTHZ-002
    note: Limit which identities may use each key, and for what.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Key_Management_Cheat_Sheet.html
  - https://csrc.nist.gov/pubs/sp/800/57/pt1/r5/final
  - https://cwe.mitre.org/data/definitions/321.html
last_reviewed: "2026-11-08"
---

Treat every key as having a life: it is generated, put into service, used for a bounded time and amount of data, rotated, and eventually retired and destroyed. Plan each stage before the first key is created, because a key that cannot be replaced turns any exposure into a permanent breach. A key compiled into code is the typical failure ([[CRYPTO-003]]).

## Practices

- **Generate** keys with a vetted library or key management service using a secure random source ([[CRYPTO-007]]). Do not derive keys from passwords or predictable values except with a purpose-built key derivation function.
- **Separate by purpose.** Use different keys for encryption, signing, token issuance and each environment or tenant that warrants it, so one compromise does not expose everything. Do not reuse one key for several algorithms.
- **Keep keys apart from the data.** Prefer a key management service or hardware-backed store where the application requests operations rather than reading raw key bytes. Envelope encryption helps: a key-encryption key protects per-dataset data keys, so rotating the top key does not require re-encrypting all data.
- **Limit use.** Restrict who and what may use or manage each key ([[AUTHZ-002]]), and log key usage and administration.
- **Rotate** on a defined schedule, whenever personnel with access leave, and immediately after suspected compromise. Support more than one active key version at a time (store a key identifier with each ciphertext or signature) so rotation does not need downtime. Take lifetime limits from NIST SP 800-57 and algorithm guidance.
- **Revoke and recover.** Document how to disable a compromised key, re-issue, and re-encrypt or re-sign affected data. Test the procedure; an untried recovery plan often fails.
- **Destroy** keys when retired, including copies and backups, once nothing needs the data they protect. Note that destroying a key also destroys access to data encrypted under it, so align with retention ([[DATA-004]]).
- **Back up** keys that must survive, with the same protection as the originals.

## For an AI coding agent

Do not generate or hard-code a key in code or tests meant for production; refer to a key by identifier and let the platform supply it. If the design has no rotation or compromise plan for a key, report that as an open risk ([[THREAT-005]]).
