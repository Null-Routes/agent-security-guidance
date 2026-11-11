---
id: DATA-001
title: Collect, return and retain only the data the purpose requires
type: principle
strength: strongly-recommended
evidence: established
category: data-protection
summary: Data you never collect, never return and no longer keep cannot leak. For each data element, tie collection, API responses and storage duration to a stated purpose, and drop what the purpose does not need.
lifecycle: [architecture, design, implementation, review]
standards: [CWE-200]
tags: [data-minimization, privacy, purpose-limitation, privacy-by-design, personal-data]
contexts:
  - Designing data models, forms, APIs and analytics events
  - Adding new fields, integrations or telemetry
  - Reviewing responses, exports and logs for unnecessary data
relationships:
  - type: mitigates
    target: DATA-005
    note: Less sensitive data in circulation means less to expose.
  - type: related_to
    target: API-003
    note: Excessive API responses violate minimization at the output boundary.
  - type: related_to
    target: LOG-002
    note: Logs are a common place where minimization is forgotten.
  - type: related_to
    target: DATA-004
    note: Retention limits are minimization over time.
  - type: related_to
    target: DATA-007
    note: When a value is needed by few components, reduce its exposure.
sources:
  - https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.01162020.pdf
  - https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html
  - https://owasp.org/API-Security/editions/2023/en/0xa3-broken-object-property-level-authorization/
last_reviewed: "2026-11-08"
---

Apply a simple test to every data element: what is the specific purpose, and is this element necessary for it? If not, do not collect it, do not return it, and do not keep it. A breach, misconfiguration or over-broad query can only expose what exists.

## Apply it at three points

- **Collection:** request only fields the feature needs. Prefer a derived or coarser value where it suffices (an age band or an "over 18" flag instead of a birth date; a country instead of a full address). Avoid collecting identifiers "in case they are useful later".
- **Use and disclosure:** return only fields the caller needs and is allowed to see; select explicit fields instead of whole records ([[API-003]]). Send third parties, analytics and support tools the minimum, and prefer pseudonymous identifiers ([[DATA-007]]).
- **Retention:** keep data only as long as the purpose lasts, then delete it ([[DATA-004]]). This includes logs, caches, exports and backups ([[LOG-002]]).

## In practice

- Record the purpose beside each field in the data model or schema documentation, so reviewers can challenge fields without one.
- Treat new fields, new consumers and new telemetry as design changes that need this question answered.
- Default to the narrow option: opt-in for optional data, fields omitted unless requested and permitted.
- Classify what remains and handle it by its sensitivity ([[DATA-002]]).

```
profile_response = pick(user, ["id", "display_name", "avatar_url"])   # explicit list
# not: serialize(user)  -> would also include email, birth date, internal flags
```

**Applies when** you handle any personal, confidential or business-sensitive data. Legal or contractual duties can require collecting or retaining certain data; the principle then means keeping to that minimum, not ignoring the duty.
