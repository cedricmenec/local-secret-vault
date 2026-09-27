## Context

Le SDK est un package unique (`src/core/`, `src/storage/`, `src/react/`) avec la crypto concentrée dans `src/core/crypto.ts`, la session dans `session.ts` et l'orchestration dans `vault.ts`. L'exploration a déjà identifié des findings concrets (verify hash × 1k brute-forçable, KDF non versionné, passphrase en sessionStorage). Le repo utilise déjà des Agent Skills sous `.agents/skills/` (openspec-*), donc le format SKILL.md est établi. Voir proposal.md pour la motivation.

## Goals / Non-Goals

**Goals:**
- Un skill réutilisable et relançable : l'audit doit pouvoir être rejoué à chaque release et produire des rapports comparables dans le temps.
- Une checklist de référence stable et versionnée dans le skill, alignée sur la checklist de l'utilisateur (AES-GCM, IV/salt, KDF versionné, secrets non persistés, mémoire, scénarios de cycle de vie).
- Un premier rapport daté qui sert de baseline de sécurité avant publication.

**Non-Goals:**
- Corriger les findings (verify hash faible, KDF non versionné…) — ce seront des changes séparés.
- Audit de dépendances npm (supply chain) — hors périmètre de la checklist crypto.
- Audit du code React (`src/react/`) au-delà de son usage du vault — la surface crypto est dans `core/`.

## Decisions

- **Skill = markdown pur, pas de script exécutable.** La checklist vit dans SKILL.md + un fichier de référence ; l'agent qui exécute le skill lit le code lui-même. Alternative envisagée : un script Node qui parse le code — rejeté car fragile face aux refactorings et moins adapté au jugement contextuel (ex. « le choix sessionStorage est-il assumé ? »).
- **Rapports dans `docs/security-audits/`, un fichier par date.** Les rapports s'accumulent sans s'écraser, permettant de suivre l'évolution des verdicts. Alternative : un seul fichier écrasé — rejeté, on perd l'historique.
- **Verdicts à quatre états** (conforme / non conforme / partiel / non applicable) plutôt qu'un booléen : plusieurs items sont réellement partiels (ex. effacement mémoire : références nullées mais plaintext non effaçable).
- **Sévérités : Critical / High / Medium / Low / Info.** Le verify hash × 1k est attendu en High (brute-force 600× moins cher que le KDF principal) ; le KDF non versionné en Medium (dette d'évolutivité, pas de faille active).
- **Le skill documente les limites JS** (GC, copies internes des strings) comme exigence du rapport, pas comme un finding — c'est une contrainte de plateforme à assumer, pas un défaut du code.

## Risks / Trade-offs

- [Le skill peut devenir obsolète face à l'évolution des recommandations KDF (OWASP, NIST)] → La checklist référence les seuils par valeur (600k itérations) ; un futur change peut les mettre à jour, et les rapports datés gardent la trace du seuil utilisé à chaque audit.
- [Un audit mené par un agent reste moins fiable qu'une revue humaine experte] → Le rapport cite le code examiné pour chaque verdict, permettant une vérification humaine ciblée.
- [Les findings documentés sans correction immédiate peuvent donner un faux sentiment de sécurité] → Le rapport classe explicitement les findings par sévérité et recommande des changes de suivi ; la baseline rend les régressions visibles.
