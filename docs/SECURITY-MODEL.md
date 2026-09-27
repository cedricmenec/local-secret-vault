# Security Model

This document explains what the vault protects against, and what it does not protect against. Read it before you store real secrets with this library.

## What the vault is

The vault stores your secrets (for example, API keys) in the browser. It encrypts them with AES-256-GCM. The encryption key is derived from a passphrase that you type. The passphrase is **never saved to disk**. Without the passphrase, the stored data is unreadable.

Key derivation uses PBKDF2-SHA256 with 600,000 iterations. This makes brute-force attacks on a stolen ciphertext slow and expensive.

## What the vault protects against

- **Direct read of IndexedDB or a stolen browser profile, while the vault is locked.** An attacker who copies the IndexedDB data (or the whole browser profile folder) gets only encrypted data. They cannot read the secrets without the passphrase.
- **Recovering the ciphertext without the passphrase.** The key comes from the passphrase. There is no key stored anywhere on disk.
- **Accidental plaintext persistence.** The vault never writes your secrets in clear text to IndexedDB, localStorage, or any file.

## What the vault does NOT protect against

- **Active XSS while the vault is unlocked.** An attacker who runs JavaScript in your page can read the decrypted secrets from memory.
- **Active XSS at any time while the session cache TTL is active.** This is a different, worse threat. To make auto-unlock work, the vault stores your **passphrase in clear text in `sessionStorage`** (see below). XSS can read it at any time — even when the vault is locked — and reuse it later. This threat outlives the current unlock.
- **Malicious browser extensions** with enough permissions. An extension can often read page data and storage.
- **A compromised browser or operating system.** Malware on the machine sees everything the browser sees.
- **Keyloggers.** A keylogger records your passphrase as you type it.
- **Consumer code that logs or exfiltrates the secret after decryption.** Once your application reads a decrypted secret, your application is responsible for it.

## The session cache trade-off

To let the page auto-unlock after a reload, the vault stores the passphrase in `sessionStorage` in clear text, with a time limit (TTL). Default TTL: 60 minutes. Options: 0 (off), 15, 60, 240 minutes, or "session" (until the tab closes).

This is a deliberate trade-off: convenience (no re-typing the passphrase) against a wider attack window for XSS.

**For high-threat environments, turn the session cache off:**

```js
import { setTTLConfig } from '@activarium/local-secret-vault'

setTTLConfig(0) // Disabled — clears the cache immediately, auto-unlock never runs
```

With TTL `0`, the cache is cleared at once and the user must type the passphrase on every page load.

## The export path

`vault.export()` returns the full encrypted vault record as a plain JSON object. This is a designed feature, for backup and migration. But it means: **any code that holds a `Vault` instance can serialize the encrypted record.** Audit the places in your application where you call `export()` and where you send the result.

## Memory storage is not for production

`createMemoryStorage` keeps the encrypted record in RAM only. It exists for **tests and development**. Do not use it in production: the vault disappears on every page reload, and it gives no persistence at all.

## Summary

| Threat | Protected? |
|---|---|
| Stolen IndexedDB / browser profile (vault locked) | ✅ Yes |
| Ciphertext without the passphrase | ✅ Yes |
| Secrets written in plaintext to storage | ✅ No (never happens) |
| XSS while the vault is unlocked | ❌ No |
| XSS while the session cache TTL is active | ❌ No (passphrase readable in sessionStorage) |
| Malicious browser extension | ❌ No |
| Compromised browser or OS | ❌ No |
| Keylogger | ❌ No |
| Your own code leaking the secret after decryption | ❌ No (your responsibility) |
