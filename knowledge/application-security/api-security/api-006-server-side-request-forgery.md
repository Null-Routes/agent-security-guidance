---
id: API-006
title: Server-side request forgery
type: vulnerability
strength: strongly-recommended
evidence: established
category: api-security
summary: When a server fetches a URL or host supplied by a client, attackers can aim it at internal services, cloud metadata or local files. Restrict destinations with allow-lists and network egress controls.
lifecycle: [design, implementation, review, deployment]
standards: [CWE-918, OWASP-API-2023:API7, OWASP-Top10-2021:A10]
tags: [ssrf, api, outbound-requests, url-fetch, webhooks, metadata]
contexts:
  - Webhooks, URL previews, image or document import-by-URL
  - Proxying, PDF or screenshot rendering, feed or XML processing
  - Services running where internal networks or cloud metadata endpoints are reachable
relationships:
  - type: related_to
    target: INPUT-002
    note: Destination allow-lists are a form of allow-list validation.
  - type: related_to
    target: ARCH-003
    note: The server sits on a trust boundary between user input and internal networks.
  - type: related_to
    target: ARCH-001
    note: Application checks and network egress rules should back each other up.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html
  - https://cwe.mitre.org/data/definitions/918.html
last_reviewed: "2026-11-07"
---

## How it arises

A feature makes the server issue a request to a location the client chose: a webhook URL, an avatar URL, a "fetch this link" preview. The server sits inside a network the client cannot reach, so the request can hit internal admin interfaces, databases, other tenants' services, or the cloud instance metadata service that hands out credentials. Responses, timing or error differences may reveal results even when the body is not returned.

## How to recognise it

- Any code that passes a user-supplied URL, hostname, IP or file reference to an HTTP client, XML/SVG/document parser or renderer.
- Checks done on the URL string only (prefix, substring, blocklist of `localhost`).
- Following redirects automatically, or resolving a hostname once to check it and again to connect.

```text
# Vulnerable: server fetches whatever it is told
body = http_get(request.params.url)
```

## How to prevent it

- Prefer not to accept URLs. If the set of targets is known, take an identifier and map it to a fixed destination.
- Otherwise allow-list schemes, hosts and ports. Do not rely on blocklists.
- Resolve the hostname, check every resulting address against disallowed ranges (loopback, link-local, private, metadata), then connect to the checked address so DNS rebinding cannot swap it.
- Do not follow redirects, or re-validate each hop.
- Limit response size and time, and do not return raw fetched content or detailed errors to the caller.
- Enforce egress controls at the network layer and run fetchers in a segment with no internal access. Use metadata-service protections that block simple forged requests.
- Do not send internal credentials or headers on these requests.

See [[API-002]] for limiting fetch size and frequency.
