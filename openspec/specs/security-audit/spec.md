# security-audit Specification

## Purpose

Définit le processus d'audit de sécurité du repo : une checklist cryptographique de référence, un Agent Skill réutilisable pour exécuter l'audit, et un format de rapport daté et versionné. Ce capability décrit le comportement attendu du skill et de ses livrables, pas le comportement du SDK lui-même.

## Requirements

### Requirement: Agent Skill d'audit de sécurité
Le repo SHALL contenir un Agent Skill `security-audit` sous `.agents/skills/security-audit/` (SKILL.md) qui, lorsqu'il est invoqué, guide un audit cryptographique du code du SDK selon la checklist de référence.

#### Scenario: Invocation du skill
- **WHEN** un agent invoque le skill `security-audit` sur ce repo
- **THEN** le skill fournit la checklist de référence et les instructions pour examiner `src/core/crypto.ts`, `src/core/session.ts`, `src/core/vault.ts` et `src/storage/`

#### Scenario: Checklist couvre les critères cryptographiques
- **WHEN** la checklist de référence est consultée
- **THEN** elle couvre au minimum : chiffrement authentifié (AES-GCM), unicité IV et salt, KDF réel (pas de SHA-256 nu), versionnement des paramètres KDF, absence de passphrase ou clé dérivée persistée sous forme exportable, gestion mémoire à `lock()`, verrouillage par timeout, et comportement sur page reload / page close / visibility change / veille / multi-tab / changement de passphrase / suppression / échec de déchiffrement / ciphertext corrompu / upgrade de schéma IndexedDB

### Requirement: Rapport d'audit daté
Chaque exécution de l'audit SHALL produire un rapport markdown horodaté (format `YYYY-MM-DD-security-audit.md`) dans `docs/security-audits/`, avec un verdict par item de checklist (conforme / non conforme / partiel / non applicable), une sévérité par finding, et des recommandations.

#### Scenario: Rapport daté et versionné
- **WHEN** l'audit est exécuté
- **THEN** un fichier `docs/security-audits/<date>-security-audit.md` est créé, daté du jour, sans écraser les rapports précédents

#### Scenario: Verdict par item
- **WHEN** un item de la checklist est examiné
- **THEN** le rapport indique un verdict explicite (conforme, non conforme, partiel, ou non applicable) avec la référence au code examiné

### Requirement: Documentation des limites de l'effacement mémoire
Le rapport d'audit SHALL documenter explicitement que JavaScript ne permet pas de garantir l'effacement physique immédiat de toutes les copies mémoire (garbage collector, copies internes des strings), même lorsque les références sont nullées à `lock()`.

#### Scenario: Section limites mémoire
- **WHEN** le rapport est généré
- **THEN** il contient une section dédiée aux limites de l'effacement mémoire en JavaScript, mentionnant le garbage collector et l'impossibilité d'effacer les copies internes des strings

### Requirement: Audit en lecture seule
L'exécution du skill d'audit SHALL être en lecture seule sur le code du SDK : elle ne modifie aucun fichier de `src/`, et se limite à créer le skill lui-même et les rapports dans `docs/security-audits/`.

#### Scenario: Aucune modification du SDK
- **WHEN** l'audit est exécuté
- **THEN** aucun fichier sous `src/` n'est modifié ; seuls le skill et le rapport daté sont créés