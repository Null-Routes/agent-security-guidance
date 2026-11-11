---
id: DATA-002
title: Classify data and handle it according to its classification
type: pattern
strength: strongly-recommended
evidence: established
category: data-protection
summary: Define a small set of sensitivity levels, label each data element, and attach concrete rules for storage, access, transmission, logging and retention to each level, so handling decisions are made once and applied consistently.
lifecycle: [architecture, design, implementation, review]
standards: [NIST-FIPS-199]
tags: [data-classification, sensitivity, handling-rules, personal-data, inventory]
contexts:
  - Designing data stores, schemas and data flows
  - Deciding what may be logged, cached, exported or sent to third parties
  - Reviewing a new feature that introduces new categories of data
relationships:
  - type: supports
    target: DATA-001
    note: Knowing what is sensitive tells you where minimization matters most.
  - type: related_to
    target: LOG-002
    note: Logging rules should be derived from classification.
  - type: related_to
    target: DATA-004
    note: Retention periods are set per class.
  - type: related_to
    target: DATA-007
    note: Higher classes justify stronger exposure reduction.
sources:
  - https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.199.pdf
  - https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html
last_reviewed: "2026-11-08"
---

Decide how sensitive each kind of data is before deciding how to protect it. Without a classification, controls are applied unevenly: secrets end up in logs, test databases or analytics because nobody marked them as different from ordinary fields.

## Do this

1. **Use few levels.** Three or four are enough, for example public, internal, confidential and restricted. More levels are rarely applied correctly. Where laws or contracts define special categories (health, payment, government identifiers, children's data), map them to the highest relevant level.
2. **Label at the source.** Record the class on each field or data set in the schema, API specification or data inventory, so tooling and reviewers can see it. Unlabelled data should be treated as at least internal until reviewed.
3. **Write rules per level.** Keep them short and checkable:

| Concern | Rule to define per class |
|---------|--------------------------|
| Storage | Whether encryption at rest and field-level protection are required ([[CRYPTO-005]], [[DATA-007]]) |
| Access | Who may read or change it; how access is logged ([[AUTHZ-002]]) |
| Transmission | Transport protection and which third parties may receive it ([[CRYPTO-004]]) |
| Logging and telemetry | Allowed, masked or forbidden ([[LOG-002]]) |
| Non-production use | Allowed only in synthetic or masked form ([[DATA-003]]) |
| Retention | Maximum period and deletion method ([[DATA-004]]) |

4. **Classify derived data by what it reveals.** Combined or aggregated data can be more sensitive than its parts; a log line or cache entry containing a restricted field inherits the class.
5. **Reclassify on change.** Review classes when the data's use, volume or regulatory context changes.

## Applies when

Any system storing more than one kind of data. For very small systems, a one-page list of fields and their levels is enough. Classification is a tool for choosing controls; it does not itself protect anything.
