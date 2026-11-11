---
id: DATA-007
title: Reduce exposure of stored sensitive values by masking, tokenizing or pseudonymizing
type: pattern
strength: recommended
evidence: established
category: data-protection
summary: Most components do not need the full value of a sensitive field. Choose among omitting, masking, tokenizing, hashing and encrypting based on who needs what, so the real value lives in as few places as possible.
lifecycle: [architecture, design, implementation]
standards: [CWE-312, CWE-359]
tags: [masking, tokenization, pseudonymization, field-level-encryption, data-exposure]
contexts:
  - Displaying, exporting or logging identifiers such as card, account or national numbers
  - Sharing data with analytics, support or non-production systems
  - Designing storage for fields most services only need to reference or compare
relationships:
  - type: supports
    target: DATA-001
    note: Reduced forms of a value are a practical way to minimize.
  - type: related_to
    target: CRYPTO-005
    note: Use authenticated encryption when the original value must be recoverable.
  - type: related_to
    target: DATA-003
  - type: related_to
    target: DATA-005
  - type: related_to
    target: AUTH-001
    note: Passwords use dedicated password hashing, not the techniques here.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html
  - https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html
  - https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.01162020.pdf
last_reviewed: "2026-11-08"
---

Ask, for each consumer of a sensitive value, what it actually does with it: show, match, count, route, or process. Give it the weakest form that supports that job, and confine the original value to a small, well-protected place.

## Choosing a technique

| Need | Technique | Notes |
|------|-----------|-------|
| Component does not use the field | **Omit** | Strongest option; do not copy it at all ([[DATA-001]]) |
| Humans need to recognise the value | **Mask** (show last few characters) | Presentation only; the stored value is unchanged, so also restrict the source |
| Systems need a stable reference, not the value | **Tokenize**: replace with a random token; mapping kept in a separate, restricted vault | Token has no mathematical link to the value; compromise of the vault exposes everything |
| Join or compare records without seeing values | **Pseudonymize**: keyed hash (HMAC) with a secret key held elsewhere | Plain unkeyed hashes of low-entropy values (phone numbers, IDs) can be reversed by guessing; re-identification stays possible for key holders |
| Original must be recovered by authorized code | **Encrypt** (field-level or storage-level) | Use authenticated encryption and managed keys ([[CRYPTO-005]], [[CRYPTO-006]]) |
| Verifying a secret the system never needs back | **Password hash** | Passwords only; see [[AUTH-001]] |

## Guidance

- Decide by classification ([[DATA-002]]): higher classes justify tokenizing or field-level protection.
- Apply the reduced form before data reaches logs, analytics, support tools, exports and non-production copies ([[LOG-002]], [[DATA-003]]).
- Keep the vault or key service separate from the data store, with narrow access and auditing. Otherwise the technique just moves the problem.
- Preserve only the format features that consumers genuinely need; each preserved property leaks information.
- Pseudonymized data is still personal data when it can be linked back; treat it accordingly.
- Test that the original value does not persist in indexes, caches, logs or backups.

```
display(card)  -> "**** **** **** 1234"        # mask
store(card)    -> token_vault.put(card) = "tok_8f3a..."   # tokenize
join_key(email)-> hmac(secret_key, lowercase(email))      # pseudonymize
```
