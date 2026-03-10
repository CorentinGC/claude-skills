---
name: pr-review
description: Pipeline de review multi-agent pour une PR — sécurité, qualité, DB, perf — avec rapport consolidé et corrections optionnelles. Utiliser /pr-review ou /pr-review <numéro-pr>.
user-invocable: true
argument-hint: [numéro-pr]
---

# Skill : PR Review Pipeline

Lance plusieurs subagents en parallèle pour reviewer une PR, puis consolide les résultats et propose des corrections.

## Argument

`$ARGUMENTS` = numéro de PR (optionnel — si absent, détecter la PR de la branche courante)

## Étape 0 — Identifier la PR et le contexte projet

Si `$ARGUMENTS` est vide, exécuter :
```bash
git branch --show-current
gh pr list --head <branche-courante> --json number,title,url --jq '.[0]'
```

Si aucune PR n'est trouvée, demander le numéro à l'utilisateur.

Stocker le numéro de PR dans `PR_NUMBER`.

Récupérer le diff complet :
```bash
gh pr diff $PR_NUMBER
```

Récupérer les métadonnées :
```bash
gh pr view $PR_NUMBER --json title,body,baseRefName,headRefName,files
```

**Contexte projet** : lire les fichiers de conventions du projet s'ils existent :
- `CLAUDE.md` (racine) — règles du projet
- `.cursor/PROJECT_CONTEXT.md` — stack, architecture, patterns

Adapter les prompts des subagents aux conventions spécifiques du projet (stack, ORM, framework, patterns actifs).

## Étape 1 — Lancer les 4 subagents en parallèle

Lancer simultanément (en une seule invocation multi-agent via l'outil Agent) les 4 agents suivants. Passer le diff complet et le contexte projet dans chaque prompt.

### Agent 1 : Security

Prompt (adapter au projet détecté) :
```
Tu es un agent de security review.

Contexte projet :
<CONTEXT>

Analyse ce diff de PR pour détecter :
- Injections : SQL (requêtes brutes sans params bindés), XSS (dangerouslySetInnerHTML sans sanitize, innerHTML), command injection (exec/spawn avec input user)
- Auth/permissions manquantes : endpoints ou actions sans vérification de session/permissions
- Secrets exposés : tokens/clés hardcodés, URLs avec credentials, variables d'env sensibles côté client
- Inputs sans validation (schéma Zod, Yup, joi, ou équivalent)
- IDOR : accès à des ressources sans vérifier l'ownership
- CORS trop permissif, headers de sécurité manquants
- Mass assignment : body request passé directement à l'ORM

Diff à analyser :
<DIFF>

Format de réponse :
## 🔒 Security

### ❌ Problèmes détectés
- [fichier:ligne] SÉVÉRITÉ — description + correctif proposé

### ✅ OK
(si aucun problème)
```

### Agent 2 : Code Quality

Prompt (adapter au projet détecté) :
```
Tu es un agent de code quality review.

Contexte projet et conventions :
<CONTEXT>

Vérifie le respect des conventions du projet. Checks génériques (toujours vérifier) :
- Taille des fichiers : max 500 LOC (warning à 400)
- console.log, console.debug, debugger — interdit en production
- Code commenté inutile (blocs >3 lignes)
- Naming : variables/fonctions explicites, pas de noms à 1 lettre hors boucles
- DRY : code dupliqué
- Fichiers modifiés uniquement — ne pas auditer tout le codebase

Checks spécifiques au framework (adapter selon le contexte projet) :
- React : hooks rules, Server vs Client Components, patterns de mémoisation
- Imports : aliases TS si configurés, imports relatifs cohérents
- Validation des inputs utilisateur
- Patterns de composants (structure des fichiers)

Diff à analyser :
<DIFF>

Format de réponse :
## 🧹 Code Quality

### ❌ Erreurs (à corriger)
- [fichier:ligne] description + correctif

### ⚠️ Warnings (à surveiller)
- [fichier:ligne] description

### ✅ OK
(si aucun problème)
```

### Agent 3 : Database

Prompt (adapter au projet détecté) :
```
Tu es un agent de database review.

Contexte projet (ORM, DB) :
<CONTEXT>

Vérifie dans le diff :
- N+1 queries : boucle avec requête unitaire → utiliser requête batch (findMany, IN clause)
- Select * implicite : toujours sélectionner les colonnes nécessaires
- Opérations liées sans transaction
- Invalidation de cache / revalidation au mauvais endroit
- Soft delete : patterns de suppression logique respectés
- Index manquants sur colonnes filtrées fréquemment
- Requêtes raw sans paramètres bindés (risque injection)
- Migrations manquantes pour changements de schéma

Si le diff ne contient aucune requête DB / code ORM : répondre directement "✅ Aucune requête DB dans ce diff."

Diff à analyser :
<DIFF>

Format de réponse :
## 🗄️ Database

### ❌ Problèmes détectés
- [fichier:ligne] description + correctif

### ✅ OK
(si aucun problème)
```

### Agent 4 : Performance

Prompt (adapter au projet détecté) :
```
Tu es un agent de performance review.

Contexte projet :
<CONTEXT>

Vérifie dans le diff :
- Composants/code côté client inutilement (préférer le rendu serveur quand possible)
- Images sans optimisation (next/image, lazy loading, dimensions manquantes)
- Imports de librairies lourdes non tree-shakées
- Synchronisation d'états via effets au lieu d'états dérivés
- Fetches séquentiels pouvant être parallélisés (Promise.all)
- Données rechargées inutilement (manque de caching)
- Composants larges englobant des parties statiques et dynamiques (découper)
- Re-renders inutiles (props instables, objets/arrays recréés inline)

Diff à analyser :
<DIFF>

Format de réponse :
## ⚡ Performance

### ❌ Problèmes détectés
- [fichier:ligne] description + correctif

### ✅ OK
(si aucun problème)
```

**Important** : dans chaque prompt, remplacer `<DIFF>` par le diff réel et `<CONTEXT>` par les conventions du projet.

## Étape 2 — Attendre les résultats et consolider

Attendre que les 4 agents terminent, puis construire le rapport consolidé :

```
## 📋 PR Review — #PR_NUMBER : <titre>

| Agent | Statut | Problèmes |
|-------|--------|-----------|
| 🔒 Security | ✅/❌ | N |
| 🧹 Code Quality | ✅/❌ | N |
| 🗄️ Database | ✅/❌ | N |
| ⚡ Performance | ✅/❌ | N |

---

<résultats détaillés de chaque agent>

---

### Recommandation
✅ Prêt à merger
ou
⚠️ Corrections mineures suggérées (non bloquantes)
ou
❌ Bloquant — corrections requises avant merge
```

Afficher ce rapport à l'utilisateur.

## Étape 3 — Proposition de correction (si problèmes détectés)

Si des problèmes ont été détectés, demander à l'utilisateur :

> Des problèmes ont été détectés. Voulez-vous que j'applique les corrections automatiques ?
> - **Auto-fixables** : suppression console.log/debugger, imports cassés, patterns interdits simples
> - **Nécessitent votre validation** : logique métier, schéma DB, permissions, architecture

Si l'utilisateur accepte :

### Étape 3a — Appliquer les corrections automatiques

Pour chaque problème auto-fixable identifié dans les rapports :
1. Lire le fichier concerné
2. Appliquer la correction (Edit tool)
3. Vérifier que la correction est correcte

### Étape 3b — Vérifications qualité post-fix

Lancer le linter du projet :
```bash
npm run lint 2>&1 | head -80
```

Si le projet a un skill `/pre-commit`, l'invoquer à la place.

### Étape 3c — Commit des corrections

Proposer un message de commit :
```
fix(pr-review): corrections automatiques PR #PR_NUMBER

- <liste des corrections appliquées>

Co-Authored-By: Claude <model> <noreply@anthropic.com>
```

**Attendre la confirmation explicite de l'utilisateur avant de commiter.**

## Rappels

- Ne jamais commiter sans validation explicite de l'utilisateur
- Ne pas modifier la logique métier sans confirmation
- Ne pas modifier les migrations DB ni les configs d'auth
- Tout changement architectural nécessite une discussion préalable
- Adapter les checks aux conventions spécifiques du projet (lire CLAUDE.md)
