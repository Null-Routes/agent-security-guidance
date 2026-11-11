---
id: INPUT-010
title: Handle file uploads as untrusted content
type: pattern
strength: strongly-recommended
evidence: established
category: input-validation
summary: Limit upload size, verify type by content, store files under server-generated names outside any executable or served path, and serve them with safe headers, because an uploaded file can carry malware, scripts or parser exploits.
lifecycle: [design, implementation, review, testing]
standards: [CWE-434, OWASP-ASVS:V5]
tags: [file-upload, content-type, storage, malware, quarantine]
contexts:
  - Features accepting images, documents, archives or imports from users
  - Profile pictures, attachments and bulk data import
  - Services that process or convert uploaded files
relationships:
  - type: supports
    target: INPUT-001
  - type: related_to
    target: INPUT-008
    note: Never use the supplied file name to build a storage path.
  - type: related_to
    target: API-002
    note: Upload size limits are one case of payload limits.
  - type: related_to
    target: AUTHZ-003
    note: Downloads need object-level authorization too.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html
  - https://cwe.mitre.org/data/definitions/434.html
last_reviewed: "2026-11-08"
---

An uploaded file is attacker-chosen name, size, type and bytes. Handle each property as untrusted, and make sure no part of the pipeline executes the file or lets a browser interpret it as active content.

## Do this

- **Authenticate and authorize uploads**, and limit the number of files, per-file size and total quota, enforced while streaming rather than after buffering ([[API-002]]).
- **Allow only needed types.** Check against an allow-list using the file's detected content (signature and structure), not the declared content type or the extension, which the sender controls. Check that extension and content agree.
- **Generate the stored name.** Use a random identifier; keep the original name only as inert metadata, encoded when displayed ([[INPUT-005]], [[INPUT-008]]).
- **Store safely.** Keep files outside any directory that is executed or served directly, ideally in object storage or a dedicated location with no execute permission.
- **Serve safely.** Return downloads through an authorized handler ([[AUTHZ-003]]) with an explicit content type, a forced-download disposition where inline display is not needed, and a header that disables content-type guessing. Serve user content from a separate domain so scripts in it cannot reach the main application's session.
- **Process defensively.** Image, document and archive parsers have a history of vulnerabilities. Run them in a sandbox with limits on CPU, memory and time. Re-encode images where possible, which also strips embedded payloads and metadata. Guard against archives that expand to enormous sizes.
- **Scan and quarantine** when policy requires malware scanning: keep files unavailable until scanning completes.

```
upload(request):
    authorize(caller); enforce_limits(stream)
    kind = detect_type(first_bytes); require kind in ALLOWED and matches(extension)
    key = random_id(); store(quarantine_area, key, stream)
    scan_and_process_in_sandbox(key); then publish(key)
```

Checks on file content do not replace the controls above: a valid image can still carry a hostile payload for a vulnerable parser.
