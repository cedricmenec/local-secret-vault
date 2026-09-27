# @activarium/local-secret-vault

Local-first encrypted vault SDK for browser-based applications. Encrypts and stores secrets on the device with PBKDF2 + AES-256-GCM, with a session cache, pluggable storage, and optional React bindings.

## Features

- **Framework-agnostic core** — pure TypeScript, no UI framework dependency.
- **AES-256-GCM encryption** — key derived via PBKDF2-SHA256 (600k iterations) with fast-path passphrase verification.
- **Session cache with TTL** — configurable auto-unlock (Disabled, 15 min, 1 hour, 4 hours, Session).
- **Pluggable storage** — built-in IndexedDB and in-memory plugins; implement `VaultStorage` for custom backends.
- **Optional React bindings** — `useVault` hook, `VaultGate`, `VaultCreateModal`, `VaultUnlockModal`, `VaultSettings`.
- **No server required** — all encryption happens client-side.

## Security warning

This vault protects your secrets at rest, but it cannot protect them from everything. In particular, XSS in your page can read the decrypted secrets (or the cached passphrase) in the browser. Before storing real secrets, read the full security model:

**→ [docs/security-model.md](./docs/security-model.md)** — what the vault protects against, and what it does not.

## Installation

```bash
npm install @activarium/local-secret-vault
```

Or with pnpm:

```bash
pnpm add @activarium/local-secret-vault
```

> **Note de migration (monorepo `my-prompt-manager`)** : en attendant la publication npm de ce package, le monorepo le consomme via une dépendance de fichier local (`"file:../local-secret-vault"`). Dès que la première version sera publiée sur npm, basculer cette dépendance vers la version publiée (ex. `"@activarium/local-secret-vault": "^0.1.0"`).

## Development

```bash
npm ci        # install dependencies from the lockfile
npm run lint  # ESLint
npm test      # Vitest
npm run build # build dist/ artifacts
```

## CI

A GitHub Actions workflow (`.github/workflows/ci.yml`) runs `npm ci`, `npm run lint`, `npm test`, and `npm run build` on every push and pull request targeting `main`.

## Quickstart

```ts
import { createVault } from '@activarium/local-secret-vault/core'
import { createMemoryStorage } from '@activarium/local-secret-vault/storage/memory'
import type { VaultPayloadBase } from '@activarium/local-secret-vault/core'

interface MyPayload extends VaultPayloadBase {
  version: 1
  apiKeys: Record<string, string>
}

const vault = createVault<MyPayload>({
  storage: createMemoryStorage(),
  initialPayload: { version: 1, apiKeys: {} },
})

await vault.create('my-secure-passphrase')

const payload = vault.getPayload()
payload.apiKeys['openrouter'] = 'sk-...'
await vault.persistPayload()

vault.lock()
```

## Documentation

- [Quickstart](docs/quickstart.md) — get a first result in minutes.
- [Concepts](docs/concepts.md) — understand how encryption, sessions, and storage work.
- [How-to guides](docs/how-to/README.md) — recipes for common tasks.
- [API reference](docs/api-reference.md) — every function, type, and option.

## License

MIT