## 1. Agent Skill security-audit

- [x] 1.1 Créer `.agents/skills/security-audit/SKILL.md` avec frontmatter (name, description) et instructions d'invocation de l'audit
- [x] 1.2 Créer la checklist de référence du skill couvrant : AES-GCM authentifié, unicité IV/salt, KDF réel et versionné, absence de passphrase/clé exportable persistée, gestion mémoire à `lock()`, verrouillage par timeout, scénarios de cycle de vie (reload, close, visibility change, veille, multi-tab, changement de passphrase, suppression, échec de déchiffrement, ciphertext corrompu, upgrade IndexedDB)
- [x] 1.3 Documenter dans le skill le format du rapport : chemin `docs/security-audits/YYYY-MM-DD-security-audit.md`, verdicts (conforme/non conforme/partiel/non applicable), sévérités (Critical/High/Medium/Low/Info), référence au code examiné par verdict
- [x] 1.4 Documenter dans le skill l'exigence de section « limites de l'effacement mémoire en JavaScript » (GC, copies internes des strings) et la règle lecture-seule sur `src/`

## 2. Exécution de l'audit initial

- [x] 2.1 Examiner `src/core/crypto.ts` contre les items de checklist crypto (AES-GCM, IV, salt, KDF, extractabilité des clés) et noter les verdicts
- [x] 2.2 Examiner `src/core/session.ts` et `src/core/vault.ts` contre les items de checklist session/mémoire (passphrase en sessionStorage, TTL, effacement à `lock()`, multi-tab) et noter les verdicts
- [x] 2.3 Examiner `src/storage/` (indexeddb, memory) contre les items de checklist stockage (upgrade de schéma, suppression, ciphertext corrompu) et noter les verdicts
- [x] 2.4 Rédiger `docs/security-audits/2026-09-27-security-audit.md` avec tous les verdicts, sévérités, recommandations, et la section limites de l'effacement mémoire

## 3. Validation

- [x] 3.1 Vérifier qu'aucun fichier sous `src/` n'a été modifié (audit en lecture seule)
- [x] 3.2 Valider le change : `openspec validate --change add-security-audit-skill`
