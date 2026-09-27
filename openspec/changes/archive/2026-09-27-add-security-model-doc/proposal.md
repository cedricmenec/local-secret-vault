## Why

The SDK makes strong security claims implicitly (PBKDF2 + AES-256-GCM, non-persisted passphrase) but never states what it does and does not protect against. A consumer could wrongly assume that secrets in IndexedDB are safe from same-origin attacks. OWASP guidance for client-side secrets is exactly this model (encrypt with a key derived from a non-persisted user passphrase), but the guarantees and — more importantly — the limits must be documented explicitly for a library of this kind.

## What Changes

- Add `docs/SECURITY-MODEL.md` (written in simplified English) describing:
  - What the vault protects against (offline read of IndexedDB / stolen browser profile while locked, ciphertext recovery without the passphrase, accidental plaintext persistence).
  - What it does NOT protect against, with two distinct XSS lines: XSS while the vault is unlocked (payload readable in memory) and XSS at any time when the session cache TTL is active (plaintext passphrase readable in `sessionStorage` — worse, it outlives the session).
  - The designed exfiltration path of `export()` (any consumer code can serialize the encrypted record).
  - Recommendation to use `setTTLConfig(0)` (Disabled) for high-threat environments.
  - A note that `createMemoryStorage` is for tests/dev only, not production.
- Add a short security warning in `README.md` linking to `docs/SECURITY-MODEL.md`.

## Capabilities

### New Capabilities

- `docs`: documentation requirements for the SDK — the security model document and the README security warning. This capability covers what documentation the package must ship and what content it must contain.

### Modified Capabilities

<!-- None: no runtime behavior of core, react, or storage-indexeddb changes. -->

## Impact

- **New files**: `docs/SECURITY-MODEL.md`.
- **Modified files**: `README.md` (security warning section).
- **Code**: none — documentation only.
- **Specs**: new `docs` capability spec; no changes to `core`, `react`, or `storage-indexeddb`.
