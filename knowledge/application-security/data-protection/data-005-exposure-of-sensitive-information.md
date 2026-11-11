---
id: DATA-005
title: Exposure of sensitive information
type: vulnerability
strength: strongly-recommended
evidence: established
category: data-protection
summary: Sensitive data leaks through many channels besides the database, including responses, logs, errors, URLs, caches, backups, client storage and third-party services. Check each channel and prevent it at the source.
lifecycle: [design, implementation, review, testing, deployment]
standards: [CWE-200, OWASP-Top10-2021:A02]
tags: [information-exposure, data-leak, sensitive-data, caches, third-parties]
contexts:
  - Reviewing how personal, financial or secret data flows through an application
  - Adding logging, analytics, caching, exports or integrations
  - Investigating a reported leak
relationships:
  - type: related_to
    target: API-003
    note: Excessive API responses are one exposure channel.
  - type: related_to
    target: LOG-002
    note: Sensitive values in logs are another.
  - type: related_to
    target: LOG-005
    note: Error messages and stack traces are another.
  - type: related_to
    target: DATA-006
    note: Values in URLs spread through logs, history and referrers.
  - type: related_to
    target: DATA-001
    note: The most reliable fix is to hold and move less data.
sources:
  - https://cwe.mitre.org/data/definitions/200.html
  - https://owasp.org/Top10/A02_2021-Cryptographic_Failures/
  - https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html
last_reviewed: "2026-11-08"
---

Exposure of sensitive information means data reaching someone who should not have it, without an exploit being required. It usually results from a missing decision about one of the places data travels or rests, not from a clever attack.

## Channels to check

| Channel | What to look for | See |
|---------|------------------|-----|
| API responses and UI | Whole records serialized; hidden fields still sent | [[API-003]] |
| Logs and metrics | Credentials, tokens, personal data in events | [[LOG-002]] |
| Errors | Stack traces, queries, paths, versions | [[LOG-005]] |
| URLs | Tokens or identifiers in query strings | [[DATA-006]] |
| Caches and CDNs | Personalized or authenticated responses cached for others | caching headers |
| Browser storage | Secrets in local storage, page source, mobile app bundles | |
| Backups and exports | Unencrypted dumps, public storage buckets, shared drives | [[CRYPTO-005]] |
| Non-production | Real data in test and staging | [[DATA-003]] |
| Third parties | Analytics, error reporting, support and AI tools receiving raw data | [[DATA-001]] |
| Transit | Plain HTTP or unverified TLS | [[CRYPTO-004]] |

## Prevent it

1. Know what is sensitive ([[DATA-002]]) and where it flows; a simple data-flow list is enough to start.
2. Reduce what exists and moves ([[DATA-001]], [[DATA-007]], [[DATA-004]]).
3. Use allow-lists of fields for outputs, logs and third-party payloads, rather than removing known-bad fields.
4. Mark responses containing private data as non-cacheable by shared caches; set storage access to private by default.
5. Encrypt stored and transmitted data where its class requires it, with managed keys ([[CRYPTO-006]]).
6. Test: scan responses, logs and artifacts with sample sensitive values to see where they appear; search repositories and images for secrets.

Exposure through failed access control is a different weakness; see [[AUTHZ-004]].
