---
id: ARCH-005
title: Authenticate and authorize service-to-service calls; do not trust network location
type: pattern
strength: strongly-recommended
evidence: established
category: secure-architecture
summary: Give every internal service a verifiable identity, authenticate and authorize each call between services, and never treat "inside the network" or "behind the gateway" as proof of trust.
lifecycle: [architecture, design, implementation, deployment]
standards: [NIST-SP-800-207]
tags: [zero-trust, mtls, workload-identity, microservices, gateway, internal-api]
contexts:
  - Microservice or multi-component backends
  - API gateways or proxies that authenticate on behalf of backends
  - Internal admin or batch interfaces assumed to be private
relationships:
  - type: implements
    target: ARCH-003
    note: Each service boundary is a trust boundary that gets authentication and authorization.
  - type: supports
    target: ARCH-001
    note: Per-service checks keep one breached host or bypassed gateway from exposing everything.
  - type: related_to
    target: AUTH-007
    note: A gateway that validates tokens must not be bypassable; backends should validate or require gateway identity.
  - type: related_to
    target: API-008
    note: Mutual TLS and workload identity are the machine-caller options there.
  - type: related_to
    target: AUTHZ-002
    note: Scope each service identity to the calls it needs.
sources:
  - https://csrc.nist.gov/pubs/sp/800/207/final
  - https://cheatsheetseries.owasp.org/cheatsheets/Microservices_Security_Cheat_Sheet.html
last_reviewed: "2026-11-08"
---

Authenticate the calling service and authorize the specific call on every internal request. Network position (private subnet, cluster, VPN, allow-listed IP) is not an identity. Zero-trust architecture guidance (NIST SP 800-207) states the same premise: no implicit trust from location.

## How to apply

1. **Give each workload its own identity,** not a shared credential. Use mutual TLS with short-lived certificates or platform-issued workload identity or signed tokens, rotated automatically ([[CRYPTO-006]]).
2. **Encrypt internal traffic** with TLS ([[CRYPTO-004]]); do not exempt it as "internal".
3. **Authorize per call.** Each service decides whether this caller may perform this action ([[AUTHZ-005]]). Grant each service identity only the callees and operations it needs ([[AUTHZ-002]]).
4. **Propagate the end user separately from the calling service.** Pass user context as a verified, signed value and re-check authorization against it, so a service cannot act with ambient, broader privileges ("confused deputy") ([[AUTHZ-004]]). Never accept a user id from a caller merely because the caller is internal.
5. **Gateways are a layer, not the only one.** If a gateway validates tokens, make sure backends are reachable only through it, or validate again, or require gateway-issued identity. Otherwise anything that reaches a backend directly bypasses authentication ([[AUTH-007]]).
6. **Cover the unglamorous paths:** internal admin ports, health and metrics endpoints, queue consumers, batch jobs and debug interfaces.

```
on request:
    caller = verify_peer_identity(connection)        # mTLS / workload identity
    if not policy.allows(caller, this_service, action): deny
    user = verify_user_assertion(request.user_token) # separate, signed
    authorize(user, resource)
```

**Applies when** a system has more than one deployable component. A small single-process application has no internal calls to authenticate, but still has external trust boundaries ([[ARCH-003]]).
