---
name: beads
description: Workflow quotidien beads (bd) — applique automatiquement dans tout projet avec .beads/ ou AGENTS.md beads. Remplace TodoWrite, gere claim/create/close de taches.
---

# Skill : Beads Workflow

Ce skill s'applique **automatiquement** dans tout projet contenant un dossier `.beads/` ou un `AGENTS.md` mentionnant beads. Il remplace `TodoWrite` pour le suivi de taches multi-sessions.

## Detection automatique

Au debut de chaque session dans un projet, verifier si beads est actif :

```bash
ls .beads/ 2>/dev/null || grep -l "beads" AGENTS.md 2>/dev/null
```

Si beads est detecte → activer ce workflow pour toute la session. Ne plus utiliser `TodoWrite`.

## Workflow automatique

### 1. En debut de tache — toujours executer

```bash
bd ready --json    # Taches sans bloqueurs
```

Selectionner la tache la plus pertinente. Si aucune ne correspond, creer une nouvelle tache.

### 2. Avant de commencer — obligatoire

```bash
bd update <id> --claim
```

### 3. Taches decouvertes en cours de route

Toute sous-tache ou bug inattendu → creer immediatement dans beads, jamais en TODO markdown :

```bash
bd create "Titre" --description="..." -t task -p 2 \
  --deps discovered-from:<parent-id> --json
```

### 4. En fin de tache — obligatoire

```bash
bd close <id> --reason "Done"
```

## Reference des commandes

```bash
bd ready --json                            # Taches disponibles (par ou commencer)
bd list --json                             # Toutes les taches
bd show <id>                               # Details d'une tache
bd create "Titre" -t feature -p 1 --json   # Creer
bd update <id> --claim                     # Revendiquer
bd update <id> --title "..." --desc "..."  # Modifier
bd close <id> --reason "Done"              # Fermer
bd dep add <id-a> <id-b>                   # id-a bloque id-b
```

### Types (-t)

`bug` · `feature` · `task` · `epic` · `chore`

### Priorites (-p)

`0` critique · `1` haute · `2` moyenne (defaut) · `3` basse · `4` backlog

## Regles

1. **Toujours `--json`** pour tout usage programmatique
2. **Preferer `bd`** a `TodoWrite` si `.beads/` est present — ne jamais mixer les deux
3. **Lier les decouvertes** : `--deps discovered-from:<parent-id>`
4. **`bd ready` avant** de demander "qu'est-ce que je fais ?"
5. **Fermer avant de passer** a la suivante : `bd close <id> --reason "Done"`
6. **Une seule tache claimed** a la fois en general

## Exemple de session type

```bash
# Debut de session
bd ready --json
# → [{"id":"bd-0012","title":"Fix login bug","type":"bug","priority":1}, ...]

# Revendiquer
bd update bd-0012 --claim

# Travail...
# Decouverte d'un bug connexe → creer dans beads directement
bd create "Null pointer in auth middleware" -t bug -p 1 \
  --deps discovered-from:bd-0012 --json

# Fin
bd close bd-0012 --reason "Fixed null check in auth middleware"
```
