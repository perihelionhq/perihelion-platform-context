---
name: review-security
description: Review a code change for injection, authentication and authorization bypasses, secrets in code or logs, insecure deserialization, SSRF, path traversal, cryptographic failures, feature-gate leaks, and protections disabled in production config. Use when reviewing for security, vulnerabilities, access control, or exploitable attack paths.
---

# Review lens: Security

Review the change the way an attacker would, looking for one exploitable path through the code and tracing whether the code stops it.

## Scope

- **Injection** (OWASP A03; CWE-89, CWE-79, CWE-78) User-controlled input reaching SQL without parameterization, HTML without escaping, shell commands without argument sanitization, or templates with raw evaluation.
- **Auth and authz bypass** (OWASP A01, A07; CWE-639, CWE-352) New endpoints without authentication, ownership checks that let user A reach user B's resources, privilege escalation to admin, CSRF on state-changing operations.
- **Secrets** (CWE-798, CWE-532) Hardcoded API keys, tokens, or passwords; credentials, PII, or session tokens written to logs or error messages; secrets in URL parameters.
- **Insecure deserialization** (OWASP A08; CWE-502) Untrusted input passed to pickle, Marshal, unserialize, or parsing of executable content.
- **SSRF and path traversal** (CWE-918, CWE-22) User-controlled URLs reaching server-side HTTP clients without an allowlist; user-controlled paths reaching the filesystem without canonicalization and boundary checks.
- **Cryptographic failures** (OWASP A02; CWE-327, CWE-916, CWE-295) Passwords hashed with MD5, SHA-1, or unsalted SHA-256 instead of a purpose-built KDF; homemade crypto or ECB mode; static IVs or keys in source; TLS verification off in production paths.
- **Feature-gate leaks** (CWE-284) A flag-gated, internal, or unreleased feature made reachable without its gate: a default flipped on, a guard dropped from one call path while siblings keep it, a route registered outside the gated block.
- **Disabled protections** (CWE-942, CWE-489) A production config or code change that turns a protection off: an untrusted or reflected origin allowed with credentials, debug or verbose-error mode enabled, security middleware removed or bypassed.

## Method

For each entry point the change adds or alters, ask how you would break it, then trace untrusted data from where it enters to the dangerous sink and check what validates, escapes, or authorizes it along the way.

Compare gated or protected paths with their siblings: a check present on one route and missing on another is a finding. Disabled protections count only when the change itself turns them off on a production path.

Which inputs are trusted, and how authorization is enforced, are often written project rules. Read the `AGENTS.md` or `CLAUDE.md` chain governing the changed files, from the repository root down.

## Threshold

Hold security to a lower bar than other lenses, because a missed vulnerability is expensive. Report a dangerous pattern with critical potential impact even when you cannot fully confirm exploitability, such as input that looks user-controlled but may be validated in middleware you cannot see, and say what you could not confirm.

Do not report defense-in-depth on code that is already protected, attacks that need physical access, side channels, or local filesystem access on the server, insecure transport in dev or test config, generic hardening advice such as rate limiting or CSP without a specific exploitable gap, the absence of a protection that was never there, or attacks that need conditions you have no evidence for.

## Reporting

- Put the OWASP category or CWE in the title when one matches; the traced path, not the identifier, justifies the finding.
- Name the entry point, the path the input takes, and the sink it reaches.
- State the impact if exploited and what would close it: the parameterization, escaping, ownership check, gate, or restored protection.
