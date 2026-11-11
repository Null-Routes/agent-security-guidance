---
id: API-007
title: Maintain an API inventory and retire old versions
type: pattern
strength: recommended
evidence: established
category: api-security
summary: Keep a current list of every API, version, environment and data flow, assign owners, and decommission deprecated versions so forgotten endpoints do not stay exposed without patches or controls.
lifecycle: [design, deployment, maintenance]
standards: [OWASP-API-2023:API9]
tags: [api, inventory, versioning, deprecation, shadow-api, lifecycle]
contexts:
  - Organizations with several teams or services exposing APIs
  - APIs that keep older versions alive for existing clients
  - Test, staging and beta endpoints reachable from the internet
relationships:
  - type: related_to
    target: ARCH-007
    note: Retired-but-running versions are a common source of outdated components.
  - type: supports
    target: ARCH-002
  - type: related_to
    target: API-001
    note: Old versions often miss access controls added to newer ones.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html
last_reviewed: "2026-11-07"
---

Know what you expose. Maintain an inventory of every API host, version, environment and consumer, and remove versions and endpoints when they are no longer needed.

Old versions, forgotten debug routes, beta hosts and undocumented "shadow" endpoints rarely receive the fixes, monitoring and access checks applied to the current API, yet they often reach the same data. Attackers probe for them by trying earlier version paths and sibling hostnames.

## What to do

- Record for each API: owner, purpose, version, environment, exposure (internet, partner, internal), authentication method, data classes handled, and known consumers.
- Generate the inventory from sources of truth where possible (the interface description, gateway configuration, deployment manifests) and compare against observed traffic to find undocumented endpoints.
- Version deliberately and publish a deprecation policy: announce a retirement date, measure remaining usage, then shut the version off rather than leaving it running.
- Apply the same authentication, authorization ([[API-001]]), logging and rate limits to every live version. Do not let a legacy path bypass controls added later.
- Keep non-production environments off the public internet or behind access control, and never point them at production data ([[DATA-003]]).
- Track third-party and partner APIs you call or expose, since they carry data flows too.
- Review the inventory on a schedule and when teams change hands, so each API still has an owner.

## Applies when

More than a handful of endpoints or teams exist. A small single-service API still benefits from knowing which versions are live and removing the old ones.
