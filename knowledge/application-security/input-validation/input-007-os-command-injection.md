---
id: INPUT-007
title: OS command injection
type: vulnerability
strength: strongly-recommended
evidence: established
category: input-validation
summary: Passing untrusted values into a shell or command line lets an attacker append their own commands. Recognise shell invocations built from strings, and prefer library APIs or direct program launch with argument arrays.
lifecycle: [implementation, review, testing]
standards: [CWE-78, CAPEC-88, OWASP-Top10-2021:A03]
tags: [command-injection, shell, os-command, subprocess, argument-injection]
contexts:
  - Code that runs operating system programs, scripts or shell pipelines
  - Wrappers around image, archive, document or network tools
  - Build, deployment and automation scripts handling external values
relationships:
  - type: related_to
    target: INPUT-003
    note: Separating arguments from the command is the primary prevention.
  - type: related_to
    target: INPUT-002
  - type: related_to
    target: AUTHZ-002
    note: Run helper processes with minimal privileges to limit impact.
sources:
  - https://cwe.mitre.org/data/definitions/78.html
  - https://capec.mitre.org/data/definitions/88.html
  - https://cheatsheetseries.owasp.org/cheatsheets/OS_Command_Injection_Defense_Cheat_Sheet.html
last_reviewed: "2026-11-08"
---

OS command injection occurs when untrusted input becomes part of a command the operating system executes. Shells treat characters such as separators, pipes, substitution syntax and redirection as instructions, so a value intended as one argument can add whole commands, run with the application's privileges.

## How to recognise it

- A command string is built by concatenation or formatting and passed to a function that invokes a shell.
- Request-derived values appear in file names, hostnames, options or flags of an external tool.
- Using a shell "for convenience" (globbing, pipes) when a library could do the work.
- Argument injection: even without a shell, a value starting with a dash can be read by the program as an option rather than data.

```
# Vulnerable: shell parses the whole string
run_shell("convert " + upload_name + " out.png")

# Better: no shell, arguments passed as a list, end-of-options marker
run_program(["convert", "--", upload_path, "out.png"])
```

## Prevent it

1. **Avoid the shell and the process entirely.** Use the language's library for the task (compression, image handling, DNS, file operations) instead of launching a tool.
2. **If you must launch a program,** call it directly with a fixed executable path and an argument array, without shell interpretation ([[INPUT-003]]).
3. **Constrain values** to strict allow-lists (a known identifier format, a fixed set of options) before use ([[INPUT-002]]).
4. **Block option injection:** put a literal end-of-options marker before untrusted arguments where the tool supports one, or reject values that begin with a dash.
5. **Do not rely on escaping** shell metacharacters; rules differ across shells and platforms and are easy to get wrong.
6. **Limit impact:** run the process as a low-privilege identity with restricted file and network access, and a minimal environment ([[AUTHZ-002]]).

Where a shell is truly required, treat the surrounding code as high risk and require explicit human review.
