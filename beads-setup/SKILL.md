---
name: beads-setup
description: Installe et configure beads (bd) — issue tracker graphique pour agents IA. Setup machine (bd + dolt), init projet, et utilisation. Invoquer avec /beads-setup.
---

# Skill : Beads Setup

Ce skill installe `bd` (beads) et `dolt` sur la machine, puis initialise beads dans le projet courant.

## Quand invoquer

- `/beads-setup` — installation complète sur une nouvelle machine + init projet
- `/beads-setup init` — uniquement `bd init` dans le projet courant (binaires déjà installés)

## Procédure d'installation

### 1. Vérifier si déjà installé

```bash
bd --version
dolt version
```

Si les deux sont présents et fonctionnels, passer directement à l'étape **Init projet**.

### 2. Installer le binaire `bd`

Récupérer la dernière version disponible :

```bash
# Vérifier la dernière version
curl -s "https://api.github.com/repos/steveyegge/beads/releases/latest" | grep '"tag_name"'
```

Télécharger et installer selon l'OS :

#### macOS / Linux
```bash
VERSION="0.59.0"  # remplacer par la version trouvée
OS=$(uname -s | tr '[:upper:]' '[:lower:]')
ARCH=$(uname -m | sed 's/x86_64/amd64/;s/aarch64/arm64/')
curl -L -o /tmp/beads.zip "https://github.com/steveyegge/beads/releases/download/v${VERSION}/beads_${VERSION}_${OS}_${ARCH}.zip"
cd /tmp && unzip -o beads.zip bd
sudo mv /tmp/bd /usr/local/bin/bd
chmod +x /usr/local/bin/bd
```

#### Windows (Git Bash)
```bash
VERSION="0.59.0"  # remplacer par la version trouvée
curl -L -o /tmp/beads_windows.zip "https://github.com/steveyegge/beads/releases/download/v${VERSION}/beads_${VERSION}_windows_amd64.zip"
cd /tmp && unzip -o beads_windows.zip bd.exe

# Copier dans le dossier nodejs (dans le PATH)
cp /tmp/bd.exe "C:/Program Files/nodejs/bd.exe"

# Patcher le wrapper shell bd pour pointer sur le .exe directement
cat > "C:/Program Files/nodejs/bd" << 'EOF'
#!/bin/sh
basedir=$(dirname "$(echo "$0" | sed -e 's,\\,/,g')")
exec "$basedir/bd.exe" "$@"
EOF
```

> **Note Windows** : Le postinstall npm de `@beads/bd` est cassé (mauvais nom de binaire attendu). Utiliser l'installation manuelle ci-dessus.

### 3. Installer `dolt`

`dolt` est la base de données versionnée utilisée par beads en backend.

```bash
# Vérifier la dernière version
curl -s "https://api.github.com/repos/dolthub/dolt/releases/latest" | grep '"tag_name"'
```

#### macOS
```bash
brew install dolt
# ou manuellement :
VERSION="1.83.4"
curl -L -o /tmp/dolt.tar.gz "https://github.com/dolthub/dolt/releases/download/v${VERSION}/dolt-darwin-amd64.tar.gz"
tar -xzf /tmp/dolt.tar.gz -C /tmp
sudo mv /tmp/dolt-darwin-amd64/bin/dolt /usr/local/bin/dolt
```

#### Linux
```bash
VERSION="1.83.4"
curl -L -o /tmp/dolt.tar.gz "https://github.com/dolthub/dolt/releases/download/v${VERSION}/dolt-linux-amd64.tar.gz"
tar -xzf /tmp/dolt.tar.gz -C /tmp
sudo mv /tmp/dolt-linux-amd64/bin/dolt /usr/local/bin/dolt
```

#### Windows (Git Bash)
```bash
VERSION="1.83.4"
curl -L -o /tmp/dolt_windows.zip "https://github.com/dolthub/dolt/releases/download/v${VERSION}/dolt-windows-amd64.zip"
cd /tmp && unzip -o dolt_windows.zip "dolt-windows-amd64/bin/dolt.exe"
cp /tmp/dolt-windows-amd64/bin/dolt.exe "C:/Program Files/nodejs/dolt.exe"
```

### 4. Vérifier l'installation

```bash
bd --version
dolt version
```

### 5. Init projet

Dans le dossier racine du projet git :

```bash
bd init
```

Cela crée :
- `.beads/` — base de données locale (committé dans git)
- `AGENTS.md` — instructions pour les agents (committé dans git)

### 6. Mettre à jour le README.md du skills repo

Après création du skill, ajouter l'entrée dans le tableau du `README.md` des skills :

```markdown
| **beads-setup** | Installe et configure beads (bd + dolt) — issue tracker graphique pour agents IA. Invoke avec `/beads-setup` |
```

## Utilisation quotidienne de beads

Une fois installé, les commandes essentielles :

```bash
bd ready --json           # Tâches sans bloqueurs (par où commencer)
bd list --json            # Toutes les tâches
bd create "Titre" -t feature -p 1 --json   # Créer une tâche
bd show bd-xxxx           # Détails d'une tâche
bd update bd-xxxx --claim # Réclamer une tâche
bd close bd-xxxx --reason "Done"  # Fermer
bd dep add bd-a bd-b      # bd-a bloque bd-b
```

### Types (-t)
`bug` · `feature` · `task` · `epic` · `chore`

### Priorités (-p)
`0` critique · `1` haute · `2` moyenne (défaut) · `3` basse · `4` backlog

## Règles d'utilisation

- Utiliser `--json` pour tout usage programmatique
- Préférer `bd` à `TodoWrite` pour les tâches multi-sessions
- Lier les tâches découvertes : `--deps discovered-from:<parent-id>`
- `bd ready` avant de demander "qu'est-ce que je fais ?"
