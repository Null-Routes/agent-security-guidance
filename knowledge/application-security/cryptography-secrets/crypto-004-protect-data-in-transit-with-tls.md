---
id: CRYPTO-004
title: Protect data in transit with current TLS and validate certificates
type: pattern
strength: strongly-recommended
evidence: established
category: cryptography-secrets
summary: Carry all non-public traffic over TLS using a current protocol version, keep certificate and hostname verification on, and apply the same standard to internal service traffic and outbound calls, not just browser-facing endpoints.
lifecycle: [design, implementation, deployment, review]
standards: [CWE-319, CWE-295, OWASP-ASVS:V12, NIST-SP-800-52]
tags: [tls, https, certificates, hsts, transport-security, certificate-validation]
contexts:
  - Any client-server, service-to-service or database connection that crosses a network
  - Code that makes outbound HTTPS calls or configures an HTTP client
  - Tests or local setups tempted to disable certificate checks
relationships:
  - type: mitigates
    target: DATA-005
    note: Cleartext or unauthenticated transport exposes data to anyone on the network path.
  - type: related_to
    target: CRYPTO-006
    note: Certificates and their private keys have a lifecycle too.
  - type: related_to
    target: ARCH-005
    note: Mutual TLS is one way to authenticate service callers.
  - type: related_to
    target: CRYPTO-008
    note: Disable deprecated protocol versions and cipher suites.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Security_Cheat_Sheet.html
  - https://csrc.nist.gov/pubs/sp/800/52/r2/final
  - https://cwe.mitre.org/data/definitions/319.html
  - https://cwe.mitre.org/data/definitions/295.html
last_reviewed: "2026-11-08"
---

Use TLS for every connection that carries credentials, personal data, tokens or any non-public content, and let the client verify the server's certificate chain and hostname. Encryption without verification only protects against passive listeners; an active attacker can still impersonate the server.

## What to do

- Serve over TLS with a currently recommended protocol version, and disable older versions and weak cipher suites. Take the acceptable set from current guidance (NIST SP 800-52, OWASP TLS cheat sheet) rather than copying a list from old configuration; see [[CRYPTO-008]].
- Redirect plain HTTP to HTTPS for browser-facing sites and send an HSTS header once HTTPS is reliably available; do not serve sensitive content over both.
- Keep certificate and hostname verification enabled in every client, including internal calls, database drivers, webhooks and test code paths that could reach production. Never ship a "skip verification" flag or an accept-all trust manager.
- For internal or private endpoints, install the private certificate authority in the client's trust store instead of disabling checks. Internal networks are not a reason to send cleartext ([[ARCH-005]]).
- Obtain certificates from a trusted authority, automate renewal and monitor expiry; protect private keys as secrets ([[CRYPTO-006]]).
- Use mutual TLS where the server must authenticate the client as well, such as service-to-service calls.
- Do not place sensitive values in URLs even over TLS ([[DATA-006]]).

```text
# Wrong: verification turned off to "make it work"
client = http_client(verify_certificates = false)

# Right: trust the right issuer, keep verification on
client = http_client(trust_roots = [public_roots, internal_ca_cert])
```

## Limits

TLS protects data on the wire between endpoints only. It does not protect data at rest ([[CRYPTO-005]]), and it ends at any proxy or load balancer that terminates it, so re-encrypt the hop behind it when the segment is not trusted.
