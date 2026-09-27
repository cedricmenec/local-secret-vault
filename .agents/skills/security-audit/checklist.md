# Security Audit Reference Checklist

Every audit must evaluate each item below and record a verdict (conforme / non conforme / partiel / non applicable) with a code reference.

## 1. Cryptography (`src/core/crypto.ts`)

- [ ] **Authenticated encryption**: secrets are encrypted with AES-GCM (authenticated), not a non-authenticated mode (CBC, CTR without MAC).
- [ ] **IV uniqueness**: the IV/nonce is randomly generated per encryption with a sufficient length (12 bytes for GCM) and never reused or derived deterministically.
- [ ] **Salt uniqueness**: a fresh random salt is generated per vault/record, of sufficient length (≥ 16 bytes), never hardcoded or reused.
- [ ] **Real KDF**: key derivation uses a real KDF (PBKDF2, Argon2, scrypt) — never a bare SHA-256 hash of the passphrase.
- [ ] **KDF parameters strength**: iteration count meets current recommendations (OWASP: PBKDF2-SHA256 ≥ 600,000 iterations).
- [ ] **KDF parameter versioning**: KDF parameters (algorithm, iterations, salt) are stored with the record so they can evolve without breaking old vaults.
- [ ] **Key extractability**: derived keys are non-extractable (`extractable: false`) where the platform allows, and no raw passphrase or derived key is exported or persisted in exportable form.
- [ ] **Verify hash separation**: any passphrase-verification hash uses independent parameters and does not weaken the main KDF (a low-iteration verify hash enables offline brute-force of the passphrase).

## 2. Session & memory (`src/core/session.ts`, `src/core/vault.ts`)

- [ ] **Passphrase persistence**: the passphrase (or derived key) is not persisted in exportable form; if cached (e.g. sessionStorage), the choice is explicit, documented, and scoped to the session.
- [ ] **Session TTL**: the cached session has a configurable TTL and expires on its own.
- [ ] **Erasure at `lock()`**: `lock()` nulls references to the passphrase/derived key and clears the session cache.
- [ ] **Timeout lock**: the vault auto-locks after the configured inactivity timeout.
- [ ] **Page reload**: behavior on reload is defined (session cache survives or is invalidated deliberately).
- [ ] **Page close**: session cache is cleared or naturally expires when the tab/window closes.
- [ ] **Visibility change**: behavior on tab visibility change is defined (e.g. optional immediate lock).
- [ ] **System sleep / wake**: behavior after system sleep is defined (TTL expiry still applies).
- [ ] **Multi-tab**: concurrent tabs behave coherently (shared or isolated session, no inconsistent state).
- [ ] **Passphrase change**: changing the passphrase re-encrypts or invalidates existing records correctly and invalidates old sessions.
- [ ] **Vault deletion**: deleting the vault removes records and session state completely.
- [ ] **Decryption failure**: a wrong passphrase or failed decryption surfaces a typed error and does not leave partial state.
- [ ] **Corrupted ciphertext**: tampered/corrupted ciphertext fails authentication (GCM tag) and raises a typed error rather than returning garbage.

## 3. Storage (`src/storage/`)

- [ ] **Schema upgrade**: IndexedDB upgrades (version bump) preserve or migrate existing encrypted records safely.
- [ ] **Record removal**: removal deletes the record (and does not leave stale metadata).
- [ ] **Corrupted ciphertext handling**: a corrupted record read from storage is surfaced as an error at unlock/decrypt time, not silently ignored.
- [ ] **Backend parity**: memory and IndexedDB backends implement the same `VaultStorage` contract with equivalent security behavior.

## 4. Platform limits (mandatory report section)

Not a code finding — a constraint to document in every report:

- JavaScript cannot guarantee immediate physical erasure of memory: the garbage collector controls reclamation.
- Strings are immutable; internal engine copies of strings (passphrases handled as strings) cannot be wiped.
- Zeroing `Uint8Array` buffers helps but is best-effort only.