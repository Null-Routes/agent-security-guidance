---
id: ARCH-006
title: Verify the integrity and provenance of third-party dependencies and build artifacts
type: pattern
strength: strongly-recommended
evidence: emerging
category: secure-architecture
summary: Pin dependencies with lockfiles and hashes, fetch them from trusted sources, review new ones before adding them, and build and verify artifacts so you can tell what was built, from what, by whom.
lifecycle: [design, implementation, deployment, maintenance]
standards: [NIST-SP-800-218, OWASP-Top10-2021:A08]
tags: [supply-chain, dependencies, lockfile, provenance, signing, typosquatting, build]
contexts:
  - Adding or updating a package, image, plugin or build tool
  - Configuring CI/CD and release pipelines
  - Consuming code or models from public registries
relationships:
  - type: related_to
    target: ARCH-007
    note: Integrity checks ensure you get what you chose; patching ensures what you chose stays safe.
  - type: supports
    target: ARCH-003
    note: Every dependency and build input is a trust boundary into your code.
  - type: related_to
    target: CRYPTO-006
    note: Signing keys need lifecycle management.
sources:
  - https://csrc.nist.gov/pubs/sp/800/218/final
  - https://csrc.nist.gov/pubs/sp/800/161/r1/final
  - https://owasp.org/Top10/A08_2021-Software_and_Data_Integrity_Failures/
last_reviewed: "2026-11-08"
---

Treat every third-party package, image, plugin, build tool and CI action as code you are choosing to run with your privileges, and make that choice verifiable. The goal is to know exactly what you depend on and to detect substitution or tampering between the author and your production system.

**Evidence note:** pinning, lockfiles and trusted sources are long-standing practice. Signed provenance and attestation (signing artifacts, recording how and where they were built, verifying at deploy time) is increasingly recommended in NIST SSDF and supply-chain guidance, but tooling and ecosystem coverage are still converging, hence `emerging`.

## How to apply

1. **Pin and lock.** Commit lockfiles; resolve exact versions, ideally with content hashes, and install reproducibly from them in CI. Avoid floating ranges and "latest" for build inputs, base images and CI steps.
2. **Use trusted sources.** Prefer a small set of vetted registries, or an internal proxy or mirror. Configure internal package names so they cannot be satisfied by a public package of the same name (dependency confusion).
3. **Vet before adding.** Check the package is the intended one (name lookalikes, typosquatting), is maintained, has a sensible scope and install behaviour, and justifies its presence. Prefer fewer dependencies; an agent should not add a package for something small and well understood.
4. **Limit what builds can do.** Run builds with least privilege ([[AUTHZ-002]]), without production secrets, and disable or review install-time scripts where practical.
5. **Verify artifacts.** Verify signatures or checksums of what you download, sign what you release, and record build provenance so deployments can check the artifact came from your pipeline.
6. **Keep an inventory** (SBOM) so you can answer "are we affected?" quickly ([[ARCH-007]]).

```
install: resolve from lockfile only; fail if hash mismatch or lockfile out of date
deploy:  accept artifact only if signature and build provenance verify
```

Unreviewed dependency or CI changes in a change request deserve the same scrutiny as code changes.
