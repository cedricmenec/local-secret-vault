# Security Audit — 2026-09-27

**Scope:** `src/core/crypto.ts`, `src/core/session.ts`, `src/core/vault.ts`, `src/storage/` (indexeddb, memory)
**Method:** read-only examination against the reference checklist in `.agents/skills/security-audit/checklist.md`
**Verdicts:** conforme / non conforme / partiel / non applicable — **Severities:** Critical / High / Medium / Low / Info

---

## 1. Cryptography (`src/core/crypto.ts`)

| Item | Verdict | Severity | Code reference |
|---|---|---|---|
| Authenticated encryption (AES-GCM) | **conforme** | — | `crypto.ts` — `encrypt()` / `decrypt()` use `AES-GCM` (256-bit keys) |
| IV uniqueness | **conforme** | — | `crypto.ts` — `generateIv()` uses `crypto.getRandomValues`, 12 bytes; a fresh IV is generated per encryption (`vault.ts` `create()`, `persistPayload()`, `changePassphrase()`) |
| Salt uniqueness | **conforme** | — | `crypto.ts` — `generateSalt()` random 16 bytes; fresh salt per vault creation and per passphrase change |
| Real KDF (no bare hash) | **conforme** | — | `crypto.ts` — `deriveKey()` uses PBKDF2-SHA-256 via Web Crypto `deriveKey` |
| KDF parameter strength | **conforme** | — | `crypto.ts` — `PBKDF2_ITERATIONS = 600_000` (meets OWASP 2023 recommendation for PBKDF2-SHA-256) |
| KDF parameter versioning | **non conforme** | Medium | `crypto.ts` / `vault.ts` `create()` — `EncryptedRecord.version` exists but PBKDF2 iteration count is a module constant; parameters are not stored per-record, so they cannot evolve without breaking old vaults |
| Key extractability | **conforme** | — | `crypto.ts` `deriveRawKey()` — keys derived with `extractable: false`, usages limited to `['encrypt', 'decrypt']`; no key export path |
| Verify hash separation | **non conforme** | High | `crypto.ts` `deriveVerifyHash()` — `VERIFY_ITERATIONS = 1_000`; the verify hash is stored in the record (`vault.ts` `create()`), so an attacker with the record can brute-force the passphrase ~600× cheaper than the main KDF. The verify hash uses the same salt as the main key, so a cracked verify hash directly yields the passphrase |

### Recommendations (crypto)

1. **High** — Raise verify-hash iterations (e.g. ≥ 100k) or replace the verify-hash scheme with a full-key trial decryption; use a dedicated salt for the verify hash.
2. **Medium** — Store KDF parameters (algorithm, iterations) in `EncryptedRecord` and honor them at unlock; bump `version` when parameters change.

## 2. Session & memory (`src/core/session.ts`, `src/core/vault.ts`)

| Item | Verdict | Severity | Code reference |
|---|---|---|---|
| Passphrase persistence | **partiel** | Medium | `session.ts` `storeSessionPassphrase()` — the raw passphrase is stored in `sessionStorage` as JSON. Per-tab and cleared on tab close, but readable by any JS running in the tab (XSS exposure) and persisted in exportable string form. The choice is documented in code comments but must be explicitly assumed by consumers |
| Session TTL | **conforme** | — | `session.ts` `getTTLConfig()` / `tryGetSessionPassphrase()` — configurable TTL (0/15/60/240/-1), expiry enforced on read, expired entries cleared |
| Erasure at `lock()` | **partiel** | Low | `vault.ts` `lock()` — nulls `_cryptoKey` and `_payload` and calls `clearSessionCache()`. However the passphrase string held by the caller and internal engine copies cannot be wiped (see §4) |
| Timeout lock | **non conforme** | Medium | No inactivity timer exists: TTL is only evaluated on `tryAutoUnlock()` / page load. An unlocked vault stays unlocked indefinitely while the page stays open, regardless of TTL |
| Page reload | **conforme** | — | `sessionStorage` survives reload by design; `tryAutoUnlock()` re-validates the TTL on load (`vault.ts` `tryAutoUnlock()`) |
| Page close | **conforme** | — | `sessionStorage` is cleared by the browser when the tab closes (`session.ts` module doc) |
| Visibility change | **non applicable** | — | No visibility-change handling implemented; no requirement violated, but see recommendation below |
| System sleep / wake | **partiel** | Low | TTL is wall-clock based (`Date.now()` in `session.ts`), so it still expires after sleep — but only when the cache is next read, not proactively |
| Multi-tab | **conforme** | — | `sessionStorage` is per-tab: each tab has its own cache and TTL; no shared inconsistent state. Trade-off: unlocking in one tab does not unlock others |
| Passphrase change | **conforme** | — | `vault.ts` `changePassphrase()` — verifies current passphrase, re-derives with a new salt + IV, re-encrypts, persists. Note: the session cache is **not** refreshed with the new passphrase (stale cache until next unlock) — Low |
| Vault deletion | **conforme** | — | `vault.ts` `delete()` — removes the storage record, nulls memory state, clears the session cache |
| Decryption failure | **conforme** | — | `vault.ts` `unlock()` — wrong passphrase throws typed `WrongPassphraseError` before any state mutation; no partial state left |
| Corrupted ciphertext | **conforme** | — | GCM authentication failure in `decrypt()` propagates as a typed error path (`WrongPassphraseError`/`VaultError` family in `errors.ts`); no garbage plaintext returned |

### Recommendations (session/memory)

3. **Medium** — Implement an inactivity auto-lock timer (reset on API use, honoring the TTL config), not just TTL-on-read.
4. **Medium** — Document prominently (README/security-model) that `sessionStorage` caching exposes the passphrase to XSS in the host page; consider caching only the derived non-extractable key handle instead of the raw passphrase.
5. **Low** — Refresh or clear the session cache after `changePassphrase()`.

## 3. Storage (`src/storage/`)

| Item | Verdict | Severity | Code reference |
|---|---|---|---|
| Schema upgrade | **partiel** | Low | `indexeddb.ts` `getDb()` — `openDB(dbName, 1, { upgrade })` creates the store if missing. Version is pinned to 1; future migrations will require bumping the version and adding upgrade logic (currently no migration path beyond creation) |
| Record removal | **conforme** | — | `indexeddb.ts` `remove()` deletes the record by key; `memory.ts` `remove()` deletes the map entry. No stale metadata remains |
| Corrupted ciphertext handling | **conforme** | — | Storage returns raw records; corruption is caught at decrypt time by GCM authentication (see §2). No silent acceptance of garbage |
| Backend parity | **conforme** | — | `indexeddb.ts` and `memory.ts` both implement `VaultStorage` (`load/save/remove/exists`) with equivalent semantics |

### Recommendations (storage)

6. **Low** — When a schema migration becomes necessary, add explicit upgrade tests (fake-indexeddb) covering records written by previous versions.

## 4. Limits of memory erasure in JavaScript (mandatory section)

JavaScript cannot guarantee immediate physical erasure of all memory copies, even when references are nulled at `lock()`:

- **Garbage collector**: reclaiming memory is under the GC's control. Zeroing a `Uint8Array` (`crypto.ts` outputs) helps, but the timing of actual reclamation is not guaranteed and cannot be forced.
- **Immutable strings**: the passphrase is handled as a `string` throughout (`create()`, `unlock()`, `session.ts` cache). Strings are immutable in JS; internal copies made by the engine (and any intermediate representations) cannot be explicitly wiped.
- **Best-effort only**: `vault.ts` `lock()` nulls `_cryptoKey` and `_payload` and clears the session cache — this is best-effort hygiene, not a guarantee.

This is a platform constraint to be acknowledged, not a code defect. Consumers with extreme threat models should assume the passphrase may linger in memory until the GC runs.

## Summary

| Severity | Count | Findings |
|---|---|---|
| Critical | 0 | — |
| High | 1 | Verify hash at 1k iterations enables cheap offline brute-force |
| Medium | 3 | KDF parameters not versioned; no inactivity auto-lock; sessionStorage passphrase XSS exposure (documented but to assume) |
| Low | 3 | Best-effort memory erasure; stale session cache after passphrase change; no migration path beyond DB creation |
| Info | — | Platform limits (§4) |

**Overall:** the core cryptographic construction (AES-256-GCM, per-record random salt/IV, PBKDF2 600k, non-extractable keys) is sound. The main actionable finding is the low-iteration verify hash; the KDF versioning and auto-lock gaps are evolvability/UX-hardening debts. No files under `src/` were modified by this audit.