---
id: INPUT-008
title: Path traversal and unsafe file path handling
type: vulnerability
strength: strongly-recommended
evidence: established
category: input-validation
summary: File paths built from untrusted input can escape the intended directory via parent-directory sequences, absolute paths or links, exposing or overwriting arbitrary files. Prefer opaque identifiers, and resolve then verify containment.
lifecycle: [implementation, review, testing]
standards: [CWE-22, CAPEC-126, OWASP-Top10-2021:A01, OWASP-ASVS:V5]
tags: [path-traversal, directory-traversal, file-access, canonicalization, symlink]
contexts:
  - File download, preview, export and static content features
  - Archive extraction and file upload handling
  - Code that opens, reads, writes or deletes files using request-supplied names
relationships:
  - type: related_to
    target: INPUT-002
    note: Canonicalize before validating.
  - type: related_to
    target: INPUT-010
    note: Uploaded file names are untrusted and must not control storage paths.
  - type: related_to
    target: AUTHZ-002
    note: The process should only be able to reach the directories it needs.
sources:
  - https://cwe.mitre.org/data/definitions/22.html
  - https://capec.mitre.org/data/definitions/126.html
  - https://owasp.org/www-community/attacks/Path_Traversal
last_reviewed: "2026-11-08"
---

Path traversal occurs when a file path is constructed from untrusted input and the result points outside the directory the developer intended. Sequences that move to a parent directory, absolute paths, alternate encodings and symbolic links can all reach other files. Impact ranges from reading secrets and source to overwriting configuration or code.

## How to recognise it

- A base directory is joined with a request-supplied file name or path segment.
- The check is a textual test (such as looking for parent-directory sequences or a prefix) performed before decoding or normalization.
- Archive extraction writes entries using names taken from inside the archive.
- Download endpoints accept a path or name and stream whatever it resolves to.
- Links inside the allowed directory can point elsewhere.

```
# Vulnerable
file = open(join(BASE_DIR, request.name))

# Safer: resolved real path must remain inside the base
target = real_path(join(BASE_DIR, request.name))   # resolves .., links, normalizes
if not is_within(target, real_path(BASE_DIR)):      # compares whole path components
    reject
```

## Prevent it

1. **Avoid user-controlled paths.** Map opaque identifiers (record IDs, random keys) to files through a lookup the server controls, and generate stored names yourself.
2. **Otherwise, resolve then verify.** Fully resolve the candidate path (decoding, normalization, links) and confirm it lies inside the allowed base, comparing path components rather than string prefixes (a base of `/data/app` must not accept `/data/app-secret`).
3. **Restrict names** with an allow-list of characters and reject separators and absolute paths ([[INPUT-002]]).
4. **Handle archives** by validating each entry's resolved destination, and refuse or neutralise links, before writing.
5. **Authorize** access to the file as an object, not only its location ([[AUTHZ-003]]).
6. **Limit the process:** run with filesystem permissions covering only the needed directories, so a mistake does not expose the whole host.

Do not rely on stripping parent-directory sequences once; nested or re-encoded variants survive naive replacement.
