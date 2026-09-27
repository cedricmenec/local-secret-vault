## Context

Le repo `local-secret-vault` a été extrait du monorepo `my-prompt-manager` (voir `changes/archive/2026-09-27-extract-local-secret-vault-package/` côté monorepo). Son dossier `openspec/` est vierge : `config.yaml` est le template par défaut, `specs/` est vide. Les specs du SDK existent uniquement dans le monorepo, sous des ids préfixés `local-secret-vault-*`.

## Goals / Non-Goals

**Goals:**
- Donner à ce repo un `openspec/config.yaml` autonome avec un `context` décrivant le SDK (pas l'app hôte).
- Migrer les 3 specs SDK avec des ids raccourcis (`core`, `react`, `storage-indexeddb`), alignés sur la structure `src/` du repo.

**Non-Goals:**
- Migrer la spec app-level `local-secret-vault` (adaptateur `src/infrastructure/vault`) — elle reste au monorepo, seule à décrire le contrat de consommation.
- Migrer les changements archivés — les décisions sont déjà consolidées dans les specs.
- Modifier le contenu normatif des specs (exigences et scénarios copiés tels quels).

## Decisions

### D1 — Renommage des ids de capability (préfixe supprimé)

`local-secret-vault-core` → `core`, etc. Dans un repo qui porte déjà le nom du package, le préfixe est redondant et alourdit chaque référence (`specs/core/spec.md` vs `specs/local-secret-vault-core/spec.md`).

*Alternative considérée* : garder les ids identiques au monorepo pour faciliter les diffs croisés. Rejeté : la continuité est assurée par le contenu (copie fidèle), pas par les ids ; le renommage est fait une fois pour toutes à la naissance du repo, avant que quiconque ne référence les nouveaux ids.

### D2 — Copie fidèle du contenu, pas de réécriture

Les exigences et scénarios sont copiés verbatim dans les deltas `ADDED Requirements`. Le texte des specs ne référence pas les ids de capability en dur, donc le renommage n'exige aucune réécriture du contenu. Toute amélioration rédactionnelle serait un changement ultérieur, pas cette migration.

*Alternative considérée* : en profiter pour moderniser le wording (ex. retirer les noms de classes internes). Rejeté : mélanger migration et réécriture rend la traçabilité avec le monorepo impossible.

### D3 — La spec app reste au monorepo

`specs/local-secret-vault/spec.md` (côté monorepo) décrit `src/infrastructure/vault/index.ts`, la base `byo-prompt-manager`, l'intégration `sessionCredentials` — aucun de ces éléments n'existe ici. C'est un contrat de consommation, pas une exigence du SDK.

*Alternative considérée* : la copier comme référence. Rejeté : une spec qui ne correspond à aucun code de ce repo serait une source de confusion, et `openspec validate` la traiterait comme une exigence de ce projet.

### D4 — `context` du config.yaml décrit le SDK standalone

Tech stack réel du repo (TypeScript strict, Vite, Vitest, Web Crypto, `idb`, React en peer optionnel), conventions de nommage observées dans `src/`, et le fait que le package est publié sous `@activarium/local-secret-vault`. Le `project.md` du monorepo n'est PAS copié : il décrit l'app BYO Prompt Manager.

## Risks / Trade-offs

- [Divergence future entre les specs du monorepo et celles de ce repo] → Les specs du monorepo deviennent la référence pour l'adaptateur app ; les specs ici sont la référence pour le SDK. Si une exigence SDK change, elle change ici puis le monorepo consomme la version publiée. À noter dans le README du monorepo si nécessaire (hors scope).
- [Perte de provenance (pas d'archives migrées)] → Le lien historique reste consultable dans `my-prompt-manager/openspec/changes/archive/` ; le changement d'extraction `2026-09-27-extract-local-secret-vault-package` documente la séparation.
- [Renommage d'ids casse les références croisées] → Aucune référence croisée entre specs par id n'a été trouvée dans le contenu ; le `config.yaml` du monorepo liste les anciens ids mais reste cohérent avec ses propres répertoires.

## Migration Plan

1. Copier les 3 specs dans les deltas du changement (fait dans ce changement).
2. À l'apply : créer `openspec/specs/{core,react,storage-indexeddb}/spec.md` (via archive du changement), et écrire le `config.yaml`.
3. Rollback : supprimer les répertoires créés ; le monorepo n'est pas touché.

## Open Questions

Aucune — les décisions de périmètre (spec app, archives, nommage) ont été tranchées avec l'utilisateur avant la rédaction.