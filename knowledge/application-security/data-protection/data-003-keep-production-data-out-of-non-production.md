---
id: DATA-003
title: Keep production data out of non-production environments
type: pattern
strength: strongly-recommended
evidence: established
category: data-protection
summary: Develop and test with synthetic or properly masked data. If real data must be copied to development, test or analytics environments, it needs the same protection and access controls as production.
lifecycle: [design, implementation, testing, deployment]
standards: [CWE-200, NIST-SP-800-53:SA-3]
tags: [test-data, synthetic-data, masking, environments, staging]
contexts:
  - Seeding development, test and staging databases
  - Reproducing production bugs
  - Sharing data with vendors, analysts or machine-learning pipelines
relationships:
  - type: mitigates
    target: DATA-005
    note: Non-production systems are a common, weakly protected route to sensitive data.
  - type: supports
    target: DATA-001
  - type: related_to
    target: DATA-007
    note: Masking and pseudonymization techniques make data safer for reuse.
  - type: related_to
    target: API-007
    note: Staging and test environments should not hold production data or be left exposed.
sources:
  - https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final
  - https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html
last_reviewed: "2026-11-08"
---

Do not copy production data into environments with weaker controls. Development, test, staging and analytics systems usually have broader access, older software, relaxed monitoring and more copies and exports, so a real record there is much more likely to leak than the same record in production.

## Preferred approaches

1. **Synthetic data:** generate realistic but fictional records with a seeded generator. Cover edge cases (long names, unusual characters, empty values) deliberately.
2. **Masked or pseudonymized extracts:** if realistic distributions are needed, transform sensitive fields irreversibly before the data leaves production ([[DATA-007]]), and check that quasi-identifiers (rare values, free text, combinations) cannot re-identify people.
3. **Minimal subsets:** take the smallest slice that serves the purpose, not a full dump.

## If real data is unavoidable

Treat the environment as production for that data: same classification rules ([[DATA-002]]), encryption, access control with named individuals, audit logging, network isolation, and a retention limit with verified deletion ([[DATA-004]]). Record who approved it and why. Never copy secrets, credentials or keys across environments; each environment has its own ([[CRYPTO-002]]).

## Also check

- Bug reports, tickets, chat messages and screenshots that paste production records.
- Backups, snapshots, and logs restored into lower environments.
- Third-party tools and SaaS test accounts connected to staging.
- Fixtures committed to repositories.
- Test code that accepts a production connection string; make it impossible by configuration, not by convention.

**Applies when** any team reproduces production behaviour elsewhere. Agents should generate synthetic fixtures by default and ask before using real records.
