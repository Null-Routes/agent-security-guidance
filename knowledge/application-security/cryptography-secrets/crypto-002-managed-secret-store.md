---
id: CRYPTO-002
title: Keep secrets in a managed secret store, not in code or images
type: pattern
strength: strongly-recommended
evidence: established
category: cryptography-secrets
summary: Store API keys, passwords and private keys in a dedicated secret store, deliver them to workloads at runtime with narrow, auditable access, and rotate them. Keep them out of source, images and logs.
lifecycle: [architecture, design, implementation, deployment, maintenance]
standards: [CWE-798, OWASP-ASVS:V11, OWASP-Top10-2021:A02]
tags: [secrets, secret-management, rotation, ci-cd, containers]
contexts:
  - Applications that call databases, third-party APIs or other services
  - CI/CD pipelines and container deployments
  - Teams currently passing secrets through config files or build arguments
relationships:
  - type: related_to
    target: CRYPTO-003
    note: A secret store removes the need to embed credentials in code.
  - type: depends_on
    target: CRYPTO-006
    note: Keys held in the store still need lifecycle and rotation management.
  - type: related_to
    target: LOG-002
    note: Retrieved secrets must not then be written to logs.
  - type: supports
    target: AUTHZ-002
    note: Per-identity secret access policies apply least privilege to credentials.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html
  - https://cheatsheetseries.owasp.org/cheatsheets/Key_Management_Cheat_Sheet.html
last_reviewed: "2026-11-07"
---

Put every credential, token, private key and similar secret in a dedicated secret store (a secrets manager, cloud key vault or equivalent) and have the running workload fetch or receive it at runtime. Do not place secrets in source code, committed config, container images, build arguments or client-side bundles.

## Why

Anything in a repository or image is copied to every clone, cache, backup and registry, and cannot be reliably recalled. A central store gives one place to restrict access, audit use, rotate and revoke.

## How to apply

- Give each workload its own identity and grant it access only to the secrets it needs. Separate production secrets from development ones.
- Inject at deploy or start time. Mounted files or in-memory retrieval are generally preferred over environment variables, which are widely visible to processes, crash dumps and diagnostics. Never bake secrets into images at build time.
- Prefer short-lived, automatically issued credentials over long-lived static ones, and automate rotation (see [[CRYPTO-006]]).
- In CI/CD, treat the pipeline as production infrastructure: scope its credentials narrowly, and let deployed workloads fetch their own secrets where possible so the pipeline never handles them.
- Scan commits and CI for leaked secrets (pre-commit and server-side), and never log plaintext secrets ([[LOG-002]]).
- Log who accessed which secret and when, and alert on unusual access.
- Have a leak runbook: revoke immediately, rotate, then clean up copies. Removing a secret from the current files does not remove it from history.

## Applies when

Any system with credentials beyond a throwaway local demo. For the failure this prevents, see [[CRYPTO-003]]. The secret store's own root credentials need separate, stricter protection.
