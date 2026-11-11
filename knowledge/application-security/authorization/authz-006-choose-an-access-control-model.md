---
id: AUTHZ-006
title: "Choose an access control model (RBAC, ABAC, relationship-based, ACLs)"
type: decision-framework
strength: recommended
evidence: established
category: authorization
summary: "Pick role-based, attribute-based, relationship-based or access-list authorization by how permissions vary: by job function, by context, by relationships between users and objects, or per individual resource."
lifecycle: [architecture, design]
standards: [NIST-SP-800-162, OWASP-ASVS:V8]
tags: [rbac, abac, rebac, acl, access-control-model, policy, decision]
contexts:
  - Starting a new product or service that needs permissions
  - Outgrowing hard-coded role checks
  - Adding sharing, delegation or multi-tenant features
relationships:
  - type: related_to
    target: AUTHZ-005
    note: Whatever model you choose, evaluate it in one enforcement point.
  - type: related_to
    target: AUTHZ-002
  - type: related_to
    target: AUTHZ-003
    note: Roles alone never answer object-level questions.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html
  - https://csrc.nist.gov/pubs/sp/800/162/upd2/final
last_reviewed: "2026-11-08"
---

Choose the simplest model that expresses your real rules, and keep the decision in one place ([[AUTHZ-005]]). Models can be combined; most systems use roles for coarse function access plus a per-object or relationship rule.

| Model | Permissions depend on | Fits when | Watch out for |
|-------|-----------------------|-----------|---------------|
| **RBAC** (role-based) | The subject's role(s) | Stable job functions, small number of roles, internal or admin tooling | Role explosion; roles cannot say "only your own records" |
| **ABAC** (attribute-based) | Attributes of subject, resource, action and environment (department, sensitivity, time, location) | Rules vary by context or data classification; regulatory conditions | Policy complexity; harder to audit "who can access X"; needs reliable attributes |
| **Relationship-based** (ReBAC) | Relationships between subject and object (owner, member of group, editor of the parent folder) | Sharing, hierarchies, collaboration products, per-object permissions at scale | Newer to many teams and tooling is still converging (evidence is less settled than for RBAC); needs a consistent relationship store |
| **ACLs** | An explicit list per resource | Few resources with individual sharing, file-style permissions | Does not scale to many objects; hard to review in aggregate |

## How to decide

1. List the questions you must answer, for example "can a manager see their reports' records?" or "can a user share a document with a colleague?".
2. If answers depend only on job function, start with RBAC.
3. If they depend on ownership or sharing, add object-level rules ([[AUTHZ-003]]) and consider relationship-based checks.
4. If they depend on data classification, device, time or region, use attributes.
5. For multi-tenant systems, tenant membership comes first, then the model within a tenant ([[AUTHZ-007]]).

Do not mix models inconsistently across services, and keep grants narrow whichever you use ([[AUTHZ-002]]). Avoid a hand-built engine if a maintained policy library or service fits.
