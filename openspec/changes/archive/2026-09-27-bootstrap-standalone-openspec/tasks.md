## 1. Bootstrap config.yaml

- [x] 1.1 Écrire `openspec/config.yaml` : remplacer le template par un `context` décrivant le SDK standalone (TypeScript strict, Vite, Vitest, Web Crypto PBKDF2 + AES-256-GCM, `idb`, React en peer optionnel, publication npm `@activarium/local-secret-vault`) et la liste des specs `core`, `react`, `storage-indexeddb`.

## 2. Migration des specs

- [x] 2.1 Créer `openspec/specs/core/spec.md` : copie fidèle du contenu de `my-prompt-manager/openspec/specs/local-secret-vault-core/spec.md` (Purpose + Requirements + Scenarios, sans les marqueurs de delta).
- [x] 2.2 Créer `openspec/specs/react/spec.md` : copie fidèle de `local-secret-vault-react/spec.md`.
- [x] 2.3 Créer `openspec/specs/storage-indexeddb/spec.md` : copie fidèle de `local-secret-vault-storage-indexeddb/spec.md`.

## 3. Validation

- [x] 3.1 Vérifier que les 3 specs migrées sont cohérentes avec le code de `src/` (core/, react/, storage/) — mêmes exports, mêmes comportements décrits.
- [x] 3.2 Exclure explicitement la spec app-level `local-secret-vault` (adaptateur) : confirmer qu'elle reste dans le monorepo et n'est pas recréée ici.