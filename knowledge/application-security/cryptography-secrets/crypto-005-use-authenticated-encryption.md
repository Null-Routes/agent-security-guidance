---
id: CRYPTO-005
title: Use authenticated encryption for data at rest and in messages
type: pattern
strength: strongly-recommended
evidence: established
category: cryptography-secrets
summary: Encrypt stored data and application-level messages with an authenticated encryption (AEAD) construction from a vetted library, never reuse a nonce under the same key, and keep keys separate from the data they protect.
lifecycle: [design, implementation, review]
standards: [CWE-311, CWE-327, CWE-323, OWASP-ASVS:V11, OWASP-Top10-2021:A02]
tags: [encryption, aead, nonce, data-at-rest, integrity, authenticated-encryption]
contexts:
  - Encrypting fields, files or backups before storage
  - Encrypting messages or tokens exchanged between components outside TLS
  - Reviewing code that uses encryption modes or padding directly
relationships:
  - type: mitigates
    target: DATA-005
    note: Encrypting stored sensitive values limits exposure if storage or backups leak.
  - type: depends_on
    target: CRYPTO-006
    note: Encryption is only as strong as key management.
  - type: related_to
    target: CRYPTO-001
    note: A concrete application of using vetted, high-level constructions.
  - type: related_to
    target: DATA-007
    note: First decide whether the value needs to be stored at all, or can be masked or tokenized.
  - type: related_to
    target: CRYPTO-007
    note: Keys and nonces come from a secure random source.
  - type: related_to
    target: CRYPTO-008
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html
  - https://csrc.nist.gov/pubs/sp/800/38/d/final
  - https://cwe.mitre.org/data/definitions/311.html
last_reviewed: "2026-11-08"
---

When data must be encrypted, use an authenticated encryption scheme (AEAD) that provides both confidentiality and tamper detection, taken from a maintained library through its high-level interface ([[CRYPTO-001]]). Encryption without authentication lets an attacker modify ciphertext undetected, and in some modes lets them recover plaintext by observing how the system reacts to altered input.

## What to do

- Choose a current AEAD construction recommended by NIST or the OWASP cheat sheet; do not combine an unauthenticated cipher mode with a hand-made integrity check. Avoid modes without authentication, and see [[CRYPTO-008]] for what to retire.
- Generate a fresh, unique nonce or IV for every message under the same key, as the construction requires, and store it with the ciphertext. Reusing a nonce under one key can destroy the security of some AEAD schemes; follow the library's nonce guidance, including limits on how much data one key may protect.
- Take keys and nonces from a secure random source where the construction calls for random values ([[CRYPTO-007]]).
- Bind context with associated data: include the record identifier, tenant or purpose so a ciphertext cannot be moved to another record and still decrypt.
- Keep keys in a key management service or secret store, not beside the data, and plan rotation ([[CRYPTO-006]], [[CRYPTO-002]]).
- Treat decryption failure as a hard failure; do not return partial plaintext or distinguishable error detail ([[LOG-005]]).
- Store an algorithm and key version with the ciphertext so you can migrate later.

```text
blob = encrypt_aead(key, plaintext, associated_data = record_id)   # library generates nonce
store(blob.version, blob.nonce, blob.ciphertext_and_tag)
plaintext = decrypt_aead(key, blob, associated_data = record_id)   # fails if altered or moved
```

## What to encrypt

Encryption is a control for stored copies you must keep. First ask whether you need the value at all ([[DATA-001]]), or whether it can be masked or tokenized instead ([[DATA-007]]). Encrypting a database column also does not help if the application decrypts it for any caller who asks, so pair it with authorization ([[AUTHZ-003]]). Passwords are not encrypted; they are hashed ([[AUTH-001]]).
