---
name: security-audit
description: Run a repeatable cryptographic security audit of the local-secret-vault SDK. Examines src/core/crypto.ts, src/core/session.ts, src/core/vault.ts and src/storage/ against a reference checklist (AES-GCM, IV/salt uniqueness, KDF strength and versioning, secret persistence, memory handling, lifecycle scenarios), then writes a dated report to docs/security-audits/. Use when asked to audit vault security, verify crypto guarantees, or produce a security baseline before a release.
---

# Security Audit

Perform a read-only cryptographic security audit of the SDK and produce a dated report.

## Invocation

When this skill is invoked:

1. Read the reference checklist in `checklist.md` (same directory as this file).
2. Examine the following files against the checklist:
   - `src/core/crypto.ts` — crypto items (AES-GCM, IV, salt, KDF, key extractability)
   - `src/core/session.ts` and `src/core/vault.ts` — session/memory items (passphrase caching, TTL, `lock()` erasure, multi-tab)
   - `src/storage/` (indexeddb, memory) — storage items (schema upgrade, removal, corrupted ciphertext)
3. For every checklist item, record an explicit verdict with a reference to the code examined (file + symbol/lines).
4. Write the report (see Report format below).
5. Do NOT modify any file under `src/`. The audit is strictly read-only on SDK code; the only outputs are this skill's own files and the dated report.

## Report format

- Path: `docs/security-audits/YYYY-MM-DD-security-audit.md` (today's date). Never overwrite a previous report — one file per date.
- One section per checklist area; each item gets a verdict:
  - **conforme** — requirement met
  - **non conforme** — requirement not met
  - **partiel** — met with caveats
  - **non applicable** — not relevant to this codebase (justify)
- Each finding gets a severity: **Critical / High / Medium / Low / Info**.
- Each verdict cites the code examined (file path and function/symbol).
- End with recommendations, ordered by severity, suggesting follow-up changes where applicable.

## Mandatory section: limits of memory erasure in JavaScript

Every report MUST contain a dedicated section documenting that JavaScript cannot guarantee immediate physical erasure of all memory copies, even when references are nulled at `lock()`:

- The garbage collector decides when memory is actually reclaimed; zeroing a `Uint8Array` helps but timing is not guaranteed.
- Strings are immutable: internal copies created by the JS engine (and any intermediate string conversions) cannot be explicitly wiped.
- This is a platform constraint to be acknowledged, not a code defect — state it as such in the report.

## Read-only rule

The audit must never change SDK behavior: no edits under `src/`, no dependency changes. Findings are documented in the report; fixes belong to separate, dedicated changes.