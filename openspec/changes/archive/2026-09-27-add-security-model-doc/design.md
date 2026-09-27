## Context

The SDK encrypts secrets with AES-256-GCM using a key derived (PBKDF2-SHA256, 600k iterations) from a user passphrase that is never persisted. However, the session cache feature stores the **plaintext passphrase** in `sessionStorage` (see `src/core/session.ts`) with a configurable TTL (default 60 min, presets 0/15/60/240/-1). The existing docs (`docs/quickstart.md`, `docs/concepts.md`, `docs/api-reference.md`) describe usage but never state the security guarantees or their limits. See proposal.md — Why for motivation.

## Goals / Non-Goals

**Goals:**
- Give consumers an honest, explicit statement of the threat model: what the vault protects against and what it does not.
- Distinguish the two XSS threats (unlocked vault vs. active session cache TTL) since they have different severity and different mitigations.
- Point high-threat consumers to the existing mitigation (`setTTLConfig(0)`).
- Warn against production use of `createMemoryStorage`.

**Non-Goals:**
- No code changes — no new APIs, no behavior changes to core/react/storage.
- No redesign of the session cache (e.g., storing a derived key instead of the passphrase) — that would be a separate change if ever pursued.
- No full security audit or penetration testing claims.

## Decisions

### D1 — Separate `docs/SECURITY-MODEL.md` rather than a README section

A dedicated document can carry the full two-list structure and the export()/TTL/memory-storage nuances without bloating the README. The README gets only a short warning linking to it.

*Alternative considered*: put everything in the README. Rejected: the README is the landing page; a detailed threat model there would bury the quickstart and be harder to maintain.

### D2 — Simplified English

The user requested simplified English. Short sentences, common vocabulary, no idioms. This also helps non-native readers, who are a large share of npm consumers.

*Alternative considered*: French (author's language) or full technical English. Rejected per explicit user choice; simplified English maximizes reach.

### D3 — Two distinct XSS lines in the "does not protect" list

XSS while unlocked exposes the decrypted payload in memory (bounded by the unlock window). XSS while TTL ≠ 0 exposes the **plaintext passphrase** in `sessionStorage` at any time — strictly worse, since it can be replayed later. Conflating them would understate the second threat.

*Alternative considered*: a single "XSS breaks everything" line. Rejected: loses the actionable distinction (mitigation for the second is `setTTLConfig(0)`).

### D4 — Document `export()` as a designed path, not a vulnerability

`export()` is a feature (backup/migration). The doc frames it as "any code holding a Vault instance can serialize the encrypted record" so consumers audit their own call sites, without implying a flaw.

### D5 — New `docs` capability

Documentation requirements are behavior consumers rely on (the package ships these docs), so they get a spec rather than `skip_specs: true`. A new `docs` capability keeps `core`/`react`/`storage-indexeddb` untouched.

*Alternative considered*: `skip_specs: true` with no delta. Rejected: the shipped docs are a verifiable deliverable; a spec makes their required content testable.

## Risks / Trade-offs

- [Doc drift: code changes (e.g., new TTL preset) make the doc stale] → The spec scenarios name the concrete facts to keep in sync; future changes touching session/cache/export must check `docs/SECURITY-MODEL.md`.
- [Overpromising by omission] → The "does not protect" list is deliberately longer than the "protects" list; review both lists together when editing.
- [Simplified English loses precision] → Keep crypto terms (PBKDF2, AES-256-GCM, ciphertext, sessionStorage) as-is; simplify only the prose around them.

## Migration Plan

1. Write `docs/SECURITY-MODEL.md`.
2. Add the README warning section with a link.
3. Rollback: delete the two edits; no code or data affected.

## Open Questions

None — scope, language, and content structure were confirmed with the user during exploration.
