---
name: self-update
description: Met a jour automatiquement les skills Claude depuis le repo git distant. Applique en debut de session si le repo est en retard sur origin.
---

# Skill : Self-Update

Ce skill s'applique automatiquement en debut de session. Il maintient les skills a jour en synchronisant avec le repo git distant.

## Procedure

En debut de session, executer silencieusement :

```bash
cd ~/.claude/skills && git fetch origin --quiet 2>/dev/null
```

Si le repo local est en retard sur `origin/main` :

```bash
cd ~/.claude/skills && git pull origin main --quiet
```

### Regles

1. **Silencieux** — Ne pas notifier l'utilisateur sauf si le pull echoue ou s'il y a des conflits.
2. **Non-bloquant** — Si le fetch/pull echoue (pas de reseau, conflit), continuer la session normalement. Ne pas bloquer le travail.
3. **Pas de force** — Ne jamais `git reset --hard` ou `git clean`. Si des modifications locales existent, les preserver et signaler le conflit a l'utilisateur.
4. **Notification minimale** — Si des skills ont ete mis a jour, mentionner en une ligne : "Skills mis a jour (X nouveaux/modifies)."
5. **Pas de push** — Ce skill ne fait que du pull. Les contributions passent par le workflow git normal.
