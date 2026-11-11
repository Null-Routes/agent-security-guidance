---
id: CRYPTO-001
title: Use vetted cryptographic primitives and libraries; do not invent your own
type: principle
strength: strongly-recommended
evidence: established
category: cryptography-secrets
summary: Build on well-reviewed algorithms through maintained, high-level cryptographic libraries. Do not design custom ciphers, protocols or constructions, and prefer APIs that make misuse hard.
lifecycle: [architecture, design, implementation, review]
standards: [CWE-327, OWASP-ASVS:V11, OWASP-Top10-2021:A02]
tags: [cryptography, libraries, algorithms, crypto-agility]
contexts:
  - Any feature that encrypts, signs, hashes or authenticates data
  - Choosing how to protect data at rest or in transit
  - Reviewing code that composes low-level crypto operations
relationships:
  - type: related_to
    target: CRYPTO-008
    note: Vetted primitives must also be currently acceptable ones.
  - type: related_to
    target: CRYPTO-005
    note: Authenticated encryption is the concrete application of this principle for stored data.
  - type: related_to
    target: ARCH-008
    note: A secret algorithm is a form of security by obscurity.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html
  - https://csrc.nist.gov/pubs/sp/800/131/a/r2/final
last_reviewed: "2026-11-07"
---

Use standard, publicly reviewed algorithms through a maintained cryptographic library, and pick the library's highest-level API that fits the task. Do not design your own cipher, hash, MAC, key exchange, token format or protocol, and do not stitch low-level primitives together when a ready-made construction exists.

## Why

Cryptographic strength comes from years of public analysis, and most real failures are in how primitives are combined (missing authentication, reused nonces, unchecked padding), not in the primitives themselves. A homemade scheme has had none of that scrutiny. OWASP's cryptographic storage guidance treats custom algorithms as off limits.

## How to apply

- Prefer high-level, misuse-resistant interfaces (for example "encrypt this message with this key") over assembling modes, padding and MACs yourself.
- Choose algorithms and parameters from current guidance such as NIST SP 800-131A and the OWASP cheat sheets. Both change over time, so re-check rather than copying numbers from old code.
- Use the platform or a well-maintained library, keep it patched, and do not fork it to "simplify" it.
- Keep algorithm choice behind a small internal interface and store an algorithm or version identifier with the output, so you can migrate when an algorithm is weakened (crypto agility).
- Where a vetted construction does not exist for your need, treat that as a design problem to escalate to a cryptography specialist rather than something to improvise.

## Applies when

Whenever code protects confidentiality, integrity or authenticity. It does not mean every system needs bespoke cryptography; most systems need to call a standard facility correctly. See [[CRYPTO-008]] for algorithms that are vetted but no longer acceptable, and [[CRYPTO-007]] for the randomness that all of this depends on.
