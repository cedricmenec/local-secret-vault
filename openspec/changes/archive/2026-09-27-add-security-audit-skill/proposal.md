## Why

Le SDK chiffre des secrets avec la Web Crypto API, mais aucun processus reproductible ne permet de vérifier périodiquement que les garanties cryptographiques annoncées (AES-GCM authentifié, KDF réel, IV uniques, effacement mémoire) tiennent face à un examen adversarial. Un audit de sécurité ponctuel vieillit mal : le code évolue, les recommandations KDF aussi. Un Agent Skill réutilisable dans `.agents/` rend cet audit relançable à chaque release, et le premier rapport daté fournit la baseline de sécurité avant publication du package.

## What Changes

- Créer un Agent Skill `security-audit` sous `.agents/skills/security-audit/` (SKILL.md + checklist de référence) qui guide un audit cryptographique du repo selon les critères : chiffrement authentifié, unicité IV/salt, KDF réel et versionné, absence de secrets persistés, gestion mémoire, comportement multi-tab et scénarios de cycle de vie (reload, veille, corruption, upgrade IndexedDB…).
- Le skill produit un rapport d'audit daté et versionné (fichier markdown horodaté) avec verdict par item de checklist, sévérité et recommandations.
- Lancer l'audit sur le code actuel (`src/core/crypto.ts`, `session.ts`, `vault.ts`, `storage/`) et produire le premier rapport daté dans `docs/security-audits/`.
- Le rapport documente explicitement les limites de l'effacement mémoire en JavaScript (garbage collector, copies internes des strings).

## Capabilities

### New Capabilities

- `security-audit`: Le processus d'audit de sécurité du repo — checklist cryptographique de référence, format du rapport daté, sévérités, et emplacement des rapports. Ce capability décrit le comportement attendu du skill et de ses livrables, pas le comportement du SDK lui-même.

### Modified Capabilities

<!-- Aucun requirement du SDK (core, react, storage-indexeddb) ne change : le skill
     est un outil d'audit, il n'altère pas le comportement du vault. Les findings
     de l'audit (ex. verify hash faible, KDF non versionné) sont documentés dans
     le rapport et pourront donner lieu à des changes séparés. -->

## Impact

- **Nouveaux fichiers** : `.agents/skills/security-audit/SKILL.md`, checklist de référence, `docs/security-audits/2026-09-27-security-audit.md`.
- **Code du SDK** : aucun changement — audit en lecture seule.
- **Findings attendus dans le rapport** (identifiés en exploration, à confirmer par l'audit) : verify hash PBKDF2 × 1k brute-forçable 600× plus vite que le KDF principal ; paramètres KDF non versionnés dans `EncryptedRecord` ; passphrase persistée en sessionStorage (choix documenté mais à assumer explicitement) ; effacement mémoire non garanti (limitation JS à documenter).
- **Aucune dépendance ajoutée** ; le skill est du markdown + instructions d'agent.
