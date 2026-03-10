---
name: smart-commit
description: Génère un message de commit conventionnel à partir des changements stagés, vérifie la qualité, et crée le commit après confirmation. Utiliser /smart-commit.
user-invocable: true
argument-hint: []
---

# Skill : Smart Commit

Génère un message de commit conventionnel, valide la qualité du code, et crée le commit après confirmation explicite.

## Étape 1 — État des changements

```bash
git status --short
git diff --cached --stat
```

### Cas : rien de stagé

Si aucun fichier n'est stagé (`git diff --cached` vide) :

1. Lister les fichiers modifiés non stagés et non trackés :
   ```bash
   git diff --name-only
   git ls-files --others --exclude-standard
   ```
2. Afficher la liste et demander à l'utilisateur quels fichiers stager.
3. Attendre la réponse avant de continuer.

### Cas : fichiers stagés

Récupérer le diff complet des fichiers stagés :
```bash
git diff --cached
```

## Étape 2 — Analyser les changements

Lire le diff stagé et identifier :
- **Type** de changement : bug fix, feature, refactor, chore, docs, test, perf, ci
- **Scope** : module principal concerné (déduire du chemin des fichiers modifiés)
- **Description** : ce qui change et pourquoi (pas "what", mais "why" quand pertinent)
- **Breaking change** : oui/non

Consulter les derniers commits pour respecter le style du projet :
```bash
git log --oneline -10
```

## Étape 3 — Générer le message de commit

Appliquer le format Conventional Commits :

```
<type>(<scope>): <description courte en impératif, max 72 chars>

[corps optionnel si contexte nécessaire]
[BREAKING CHANGE: description si applicable]

Co-Authored-By: Claude <model> <noreply@anthropic.com>
```

### Types autorisés
- `feat` — nouvelle fonctionnalité
- `fix` — correction de bug
- `refactor` — refactoring sans changement de comportement
- `chore` — maintenance, dépendances, tooling
- `docs` — documentation uniquement
- `test` — ajout/modification de tests
- `perf` — optimisation de performance
- `ci` — CI/CD, scripts, configuration
- `style` — formatage, whitespace (sans changement de logique)

### Scope
Déduire du chemin des fichiers modifiés :
- `src/components/Calendar/` → `calendar`
- `src/actions/missions/` → `missions`
- `prisma/migrations/` → `prisma`
- `src/lib/auth*` → `auth`
- Plusieurs modules → scope le plus englobant, ou pas de scope si trop large

## Étape 4 — Vérifications qualité

Avant de proposer le commit, lancer le linter :

```bash
npm run lint 2>&1 | head -50
```

Si lint KO : afficher les erreurs et demander si l'utilisateur veut corriger avant de commiter.

**Ne pas lancer le build** — c'est géré par le hook pre-commit automatique s'il existe.

## Étape 5 — Proposer le commit

Afficher :

```
## Commit proposé

`<type>(<scope>): <description>`

<corps si applicable>

Fichiers inclus :
- path/to/file1.ts
- path/to/file2.tsx

Confirmer le commit ? (oui/non/modifier)
```

- **oui** → procéder au commit
- **non** → annuler
- **modifier** → demander les corrections et regénérer

## Étape 6 — Créer le commit (après confirmation)

```bash
git commit -m "$(cat <<'EOF'
<type>(<scope>): <description>

<corps si applicable>

Co-Authored-By: Claude <model> <noreply@anthropic.com>
EOF
)"
```

Vérifier le succès :
```bash
git log --oneline -1
```

Si le hook pre-commit échoue (lint/build), afficher l'erreur et proposer de corriger avant de réessayer.

## Rappels

- **JAMAIS** commiter sans confirmation explicite de l'utilisateur
- Respecter la branche protégée du projet (ne pas commiter sur main/master sans autorisation)
- Si le hook échoue : corriger et créer un NOUVEAU commit (ne pas amender)
- Consulter CLAUDE.md du projet pour les conventions spécifiques de commit
