---
id: AUTH-001
title: Hash passwords with an adaptive, salted password hashing function
type: pattern
strength: mandatory
evidence: established
category: authentication
summary: Store only salted hashes produced by a purpose-built, tunable password hashing algorithm (Argon2id, scrypt, bcrypt, PBKDF2). Never store passwords in plaintext, reversibly encrypted, or under a fast general-purpose hash.
lifecycle: [design, implementation, review, maintenance]
standards: [CWE-916, CWE-257, OWASP-Top10-2021:A02, OWASP-ASVS:V11]
tags: [password, hashing, argon2, bcrypt, scrypt, pbkdf2, salt, credentials]
contexts:
  - Application verifies user-chosen passwords itself
  - Migrating legacy password stores
relationships:
  - type: mitigates
    target: AUTH-004
    note: Slow, salted hashes limit offline cracking of a leaked credential database.
  - type: related_to
    target: CRYPTO-001
  - type: related_to
    target: CRYPTO-007
    note: Salts must come from a secure random source.
  - type: related_to
    target: AUTH-008
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html
  - https://cwe.mitre.org/data/definitions/916.html
  - https://cwe.mitre.org/data/definitions/257.html
  - https://pages.nist.gov/800-63-3/sp800-63b.html
last_reviewed: "2026-11-07"
---

Hash every password with a memory-hard or otherwise deliberately slow, salted password hashing function, and store only the resulting hash plus its parameters. Do not store passwords in plaintext, encrypt them reversibly, or use a fast hash such as a bare SHA-2 digest.

**Why:** databases leak. If an attacker obtains the stored values, the work factor of the hash decides how many guesses per second they can make offline. Plaintext or reversible storage gives them every password at once, and users reuse passwords elsewhere (see [[AUTH-004]]).

**Do this**

- Prefer Argon2id; scrypt, bcrypt and PBKDF2 with a high iteration count are acceptable when constrained by platform or compliance. Take current cost parameters from the OWASP cheat sheet rather than hard-coding values from memory.
- Use a unique random salt per password from a secure source ([[CRYPTO-007]]). Mature libraries do this for you; call their high-level API.
- Store the algorithm identifier and parameters with the hash so you can raise the cost later and rehash on the next successful login.
- Compare using the library's verify function, which is constant-time.
- Do not silently truncate long passwords; check your library's input limits (bcrypt, for example, has a byte limit) and pre-process safely or choose another algorithm.
- Optionally add a server-side secret ("pepper") held outside the database, understanding it adds key-management burden ([[CRYPTO-006]]).

```
record = { algo: "argon2id", params: {...}, salt: random(16), hash: H }
login:   ok = password_verify(record, candidate)   # library, constant time
         if ok and needs_rehash(record): store(password_hash(candidate))
```

**Why this is mandatory:** NIST SP 800-63B requires verifiers to store memorized secrets salted and hashed with an approved one-way scheme, and plaintext or reversibly encrypted storage turns any read of the database (or a backup) into a mass credential compromise. Because users reuse passwords, the damage extends to other systems. Correct hashing is available in every mainstream cryptographic library, so there is no legitimate design that needs an exception.

**Applies when** your system verifies passwords itself. If authentication is delegated to a federated provider or is fully passwordless, you hold no passwords to hash ([[AUTH-008]]).
