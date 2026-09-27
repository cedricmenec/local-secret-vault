## Why

Le projet vient d'être extrait du monorepo `my-prompt-manager` dans son propre repository. Son dossier `openspec/` est encore vierge (config template, aucune spec), alors que tout le travail de spécification du SDK a été fait dans le monorepo. Sans migration, ce repo perd la mémoire des exigences du SDK et tout futur changement OpenSpec partirait de zéro.

## What Changes

- Migrer les 3 specs SDK depuis `my-prompt-manager/openspec/specs/` vers `openspec/specs/` ici, avec renommage (le préfixe `local-secret-vault` est redondant dans un repo qui porte déjà ce nom) :
  - `local-secret-vault-core` → `core`
  - `local-secret-vault-react` → `react`
  - `local-secret-vault-storage-indexeddb` → `storage-indexeddb`
- Bootstrap `openspec/config.yaml` : `context` décrivant le SDK standalone (TypeScript strict, Vite, Vitest, Web Crypto, `idb`, React en peer optionnel) et liste des 3 specs.
- Ne PAS migrer la spec app-level `local-secret-vault` (adaptateur `src/infrastructure/vault`, base IndexedDB `byo-prompt-manager`, intégration `sessionCredentials`) : elle décrit le contrat de consommation côté monorepo et reste dans `my-prompt-manager`.
- Ne PAS migrer les changements archivés liés au vault (8 dossiers dans `changes/archive/`) : les décisions y sont déjà consolidées dans les specs ; les archives restent l'historique du monorepo.
- Côté monorepo (hors scope de ce repo, à faire là-bas) : la spec app `local-secret-vault` reste en place ; rien à modifier de son côté pour cette migration.

## Capabilities

### New Capabilities

- `core`: exigences du cœur framework-agnostic du SDK — classe `Vault<TPayload>`, cycle de vie (create/unlock/lock/delete), export/import, changement de passphrase, auto-unlock via cache de session avec TTL, interface `VaultStorage`, erreurs typées, vérification de disponibilité Web Crypto.
- `react`: exigences des bindings React optionnels — hook `useVault`, composants `VaultGate`, `VaultCreateModal`, `VaultUnlockModal`, `VaultSettings`, avec props `classNames` pour la personnalisation.
- `storage-indexeddb`: exigences du plugin de stockage IndexedDB intégré — factory `createIndexedDbStorage`, création automatique de l'object store, persistance binaire-safe des `EncryptedRecord`, isolation entre bases.

### Modified Capabilities

<!-- Aucune : ce changement ne modifie aucune exigence existante dans ce repo (aucune spec n'y existe encore). -->

## Impact

- **Ce repo** : `openspec/config.yaml` (bootstrap), `openspec/specs/{core,react,storage-indexeddb}/spec.md` (créés par migration + renommage).
- **Monorepo `my-prompt-manager`** : aucune modification requise de son côté pour ce changement ; ses 4 specs restent en l'état (dont la spec app `local-secret-vault` qui y reste pertinente).
- **Code source** : aucun impact — migration de documentation de spécifications uniquement.
- **Contenu des specs** : copie fidèle du contenu existant ; seuls les ids de capability changent (renommage de répertoire). Le texte des specs ne référence pas les ids de capability en dur, donc aucune réécriture de contenu n'est nécessaire.