## 1. Security model document

- [x] 1.1 Create `docs/SECURITY-MODEL.md` in simplified English with: intro (what the vault is, one paragraph), "What the vault protects against" list (IndexedDB/profile read while locked, ciphertext without passphrase, no accidental plaintext persistence), "What the vault does NOT protect against" list (XSS while unlocked, XSS while session TTL active — passphrase in sessionStorage, malicious extensions, compromised browser/OS, keyloggers, consumer code logging/exfiltrating after decryption), with the two XSS threats as distinct entries.

## 2. Additional security notes in the document

- [x] 2.1 Add to `docs/SECURITY-MODEL.md`: a section on `export()` as a designed data path (any code holding a `Vault` instance can serialize the encrypted record), a recommendation to use `setTTLConfig(0)` for high-threat environments (clears the cache immediately, disables auto-unlock), and a warning that `createMemoryStorage` is for tests/dev only, not production.

## 3. README warning

- [x] 3.1 Add a short security warning section to `README.md` (simplified English) linking to `docs/SECURITY-MODEL.md`.

## 4. Validation

- [x] 4.1 Verify the document content against the `docs` spec scenarios (both lists complete, two XSS lines distinct, export/TTL/memory-storage sections present, README link works).
