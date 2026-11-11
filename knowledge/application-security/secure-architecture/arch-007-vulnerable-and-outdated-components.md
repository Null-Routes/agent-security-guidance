---
id: ARCH-007
title: Vulnerable and outdated components
type: vulnerability
strength: strongly-recommended
evidence: established
category: secure-architecture
summary: Known-vulnerable, unmaintained or forgotten libraries, runtimes, images and services give attackers ready-made exploits. Keep an inventory, watch advisories, patch within defined time limits and remove what you do not use.
lifecycle: [implementation, review, deployment, maintenance]
standards: [CWE-1104, OWASP-Top10-2021:A06, NIST-SP-800-218]
tags: [dependencies, patching, sbom, sca, end-of-life, advisories, vulnerability-management]
contexts:
  - Any application using third-party libraries, frameworks, runtimes or container images
  - Long-lived systems and services nobody actively maintains
  - Responding to a newly published advisory
relationships:
  - type: related_to
    target: ARCH-006
    note: Integrity controls get you the right component; this entry keeps it up to date.
  - type: related_to
    target: API-007
    note: Retired-but-running API versions and hosts are a common outdated-component source.
  - type: related_to
    target: ARCH-002
    note: Removing unused components shrinks the attack surface.
sources:
  - https://owasp.org/Top10/A06_2021-Vulnerable_and_Outdated_Components/
  - https://cheatsheetseries.owasp.org/cheatsheets/Vulnerable_Dependency_Management_Cheat_Sheet.html
  - https://cwe.mitre.org/data/definitions/1104.html
last_reviewed: "2026-11-08"
---

Most applications are mostly third-party code. When a component has a published vulnerability or no longer receives fixes, attackers can often exploit it with public tooling and no knowledge of your application.

## How it arises

- Dependencies are added once and never updated; lockfiles freeze old versions.
- Transitive dependencies are invisible to the team.
- Base images, runtimes, operating systems and build tools age without anyone owning them.
- End-of-life software stays in service, or old API versions and hosts keep running ([[API-007]]).
- Nobody is watching advisories, or advisories are seen but not triaged.

## How to prevent and detect

1. **Inventory.** Keep a current list of components, including transitive ones and images, ideally as a machine-readable software bill of materials, tied to each deployed version.
2. **Scan and watch.** Run software composition analysis in CI and on deployed artifacts regularly, not only at build time, and subscribe to advisories for key components.
3. **Triage by exposure.** Judge severity by whether the vulnerable code path is reachable and exposed in your use, but record the decision. Do not dismiss findings merely because they are in a transitive dependency.
4. **Patch on a schedule.** Set time limits by risk (for example, faster for internet-facing and actively exploited issues), automate routine updates, and run tests so updating is cheap.
5. **Plan for end of life.** Choose maintained components, track support dates, and replace or isolate those that lose support.
6. **Remove the unused.** Delete dependencies, features and services nobody needs.
7. **If you cannot patch,** apply a compensating control (disable the feature, restrict network access, add a filter) and track it as temporary ([[ARCH-001]]).

Pair this with integrity and provenance controls ([[ARCH-006]]); updating from an unverified source trades one risk for another.
