---
id: DATA-004
title: Define retention periods and delete data when no longer needed
type: pattern
strength: recommended
evidence: established
category: data-protection
summary: Give each data class an explicit retention period tied to its purpose and legal duties, delete automatically at expiry across all copies including backups and derived stores, and verify that deletion happened.
lifecycle: [design, implementation, deployment, maintenance]
standards: [NIST-SP-800-53:SI-12]
tags: [retention, deletion, data-lifecycle, backups, legal-hold]
contexts:
  - Designing storage for user data, logs, uploads and analytics
  - Handling account closure and deletion requests
  - Setting log, backup and archive policies
relationships:
  - type: supports
    target: DATA-001
    note: Retention limits apply minimization over time.
  - type: related_to
    target: LOG-002
  - type: related_to
    target: LOG-003
    note: Log retention must balance investigation needs against holding data longer than necessary.
  - type: related_to
    target: DATA-002
    note: Retention periods are set per classification.
sources:
  - https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final
  - https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.01162020.pdf
last_reviewed: "2026-11-08"
---

Decide at design time how long each kind of data is kept, and build deletion in as a feature. Data that is kept indefinitely accumulates risk and breach impact for no benefit, and is hard to find and remove later.

## Do this

1. **Set a period per class.** Base it on the purpose and on legal, contractual and operational duties (such as audit or investigation needs); write it down with the owner. Prefer a defined end over "until further notice".
2. **Automate expiry.** Use scheduled jobs, storage lifecycle rules or time-to-live settings rather than manual cleanups. Store timestamps that make expiry calculable.
3. **Cover every copy.** Primary stores, replicas, caches, search indexes, analytics and data-warehouse extracts, uploaded files, exports, and logs. Backups cannot usually be edited, so give them a bounded retention, and make sure restores re-apply deletions made since the backup.
4. **Delete in the right way.** Remove, anonymize irreversibly, or for encrypted stores destroy the key ([[CRYPTO-006]]) when that covers all copies. Soft-delete flags alone are not deletion.
5. **Honour exceptions explicitly.** Legal holds, regulatory minimums and disputes pause deletion for scoped data only, recorded and time-limited.
6. **Verify.** Test that expired or deleted records are gone from each store, and monitor the deletion jobs so silent failures are noticed ([[LOG-001]]).
7. **Handle account closure** as a defined workflow: what is deleted immediately, what is retained and why, and for how long.

```
every day:
    for class in data_classes:
        delete_where(created_at < now - class.retention and not on_legal_hold)
    log(deleted_counts)    # no record contents
```

**Applies when** you control storage of data that outlives its purpose. Do not delete audit evidence earlier than your security and legal obligations require; the aim is a justified period, not the shortest possible.
