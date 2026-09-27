## Purpose

Defines the security documentation the SDK must ship: an explicit security model document stating what the vault protects against and what it does not, plus a README warning pointing consumers to it.

## ADDED Requirements

### Requirement: Security model document
The package SHALL ship a `docs/SECURITY-MODEL.md` document, written in simplified English, that describes the security properties of the vault. The document SHALL contain two explicit lists: what the vault protects against, and what it does not protect against.

#### Scenario: Document lists protections
- **WHEN** a reader opens `docs/SECURITY-MODEL.md`
- **THEN** the document contains a "protects against" section covering at least: direct read of IndexedDB or a stolen browser profile while the vault is locked, recovery of the ciphertext without the passphrase, and accidental persistence of plaintext secrets

#### Scenario: Document lists limitations
- **WHEN** a reader opens `docs/SECURITY-MODEL.md`
- **THEN** the document contains a "does not protect against" section covering at least: active XSS while the vault is unlocked, active XSS at any time while the session cache TTL is active (plaintext passphrase readable in sessionStorage), malicious browser extensions with sufficient permissions, a compromised browser or OS, keyloggers, and consumer code that logs or exfiltrates the secret after decryption

#### Scenario: Document distinguishes the two XSS threats
- **WHEN** a reader reads the XSS entries in the "does not protect against" section
- **THEN** XSS while the vault is unlocked (payload readable in memory) and XSS while the session cache TTL is active (passphrase readable in sessionStorage, outliving the current unlock) are described as two distinct threats

### Requirement: Security model documents the export path
The security model document SHALL state that `export()` returns the full encrypted vault record as a JSON-serialisable object, and that any consumer code holding a `Vault` instance can serialize it.

#### Scenario: Export path is documented
- **WHEN** a reader reads the security model document
- **THEN** the document mentions `export()` as a designed data path that consumer code can use to obtain the encrypted record

### Requirement: Security model recommends disabling the session cache for high-threat environments
The security model document SHALL recommend setting the session TTL to `0` (Disabled) via `setTTLConfig(0)` for high-threat environments, and SHALL state that this clears the session cache immediately and prevents auto-unlock.

#### Scenario: TTL recommendation is present
- **WHEN** a reader reads the security model document
- **THEN** the document recommends `setTTLConfig(0)` for high-threat environments
- **AND** explains that this clears the session cache immediately and disables auto-unlock

### Requirement: Memory storage is documented as non-production
The security model document SHALL state that `createMemoryStorage` is intended for tests and development only, and SHALL NOT be used in production.

#### Scenario: Memory storage warning is present
- **WHEN** a reader reads the security model document
- **THEN** the document states that `createMemoryStorage` is for tests/dev only

### Requirement: README security warning
The package README SHALL contain a short security warning section that links to `docs/SECURITY-MODEL.md`.

#### Scenario: README links to the security model
- **WHEN** a reader opens `README.md`
- **THEN** a security warning section is present
- **AND** it links to `docs/SECURITY-MODEL.md`
