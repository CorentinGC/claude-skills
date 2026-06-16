---
name: project-memory
description: Maintient un MEMORY.md a la racine de chaque projet (committable) — mémoire de travail : état courant, prochaines étapes, décisions récentes, pièges. À LIRE avant de coder, à METTRE À JOUR après chaque edit. Complémentaire à /docs + llms.txt (la doc stable).
---

# Skill : Project Memory

Ce skill garantit qu'un fichier `MEMORY.md` existe et reste à jour à la racine de chaque projet. C'est la **mémoire de travail** de l'agent, pas de la documentation.

## Rôle (et ce que ce n'est PAS)

`MEMORY.md` permet de **reprendre vite** un projet : où on en est, ce qui reste à faire, les décisions récentes, les pièges connus. C'est une mémoire court/moyen terme.

- **MEMORY.md** = état courant + reprise (volatile, change souvent). Lu avant de coder, mis à jour après chaque edit.
- **docs/ + llms.txt** = documentation stable (archi, conventions, modules). Mise à jour aux changements structurels.

Ne pas confondre ni dupliquer : si une info est stable, elle va dans `docs/` et on la référence depuis `MEMORY.md`.

## Emplacement

À la racine du projet, committable, pour que tous (humains et agents) puissent reprendre :
```
./MEMORY.md
```

## Cadence

- **Lecture** : au début de chaque session **et avant toute tâche de code**. C'est le premier fichier à consulter (avec `llms.txt`).
- **Mise à jour** : **après chaque edit significatif** — état courant, prochaines étapes, décisions, gotchas découverts. Garder le fichier dense et court (< 150 lignes).
- **Création** : si `MEMORY.md` est absent, le générer (le skill `/project-setup` le fait dans le cadre d'un setup complet ; sinon, le créer seul ici).

## Template

```markdown
# MEMORY — [Nom du projet]

> Mémoire de travail. À LIRE avant de coder, à METTRE À JOUR après chaque edit. Concis (< 150 lignes). Doc détaillée → `llms.txt` / `docs/`.

## État courant
- [Sur quoi on travaille / dernier point d'arrêt.]

## Prochaines étapes
- [ ] [...]

## Décisions récentes
- [AAAA-MM-JJ] — [décision + raison courte.]

## Pièges / gotchas
- [Ce qui surprend, contourne, ou casse facilement.]

## Pointeurs
- Archi : `docs/architecture.md` · Setup : `docs/setup.md` · Index : `llms.txt`
```

> Si le projet n'a pas de `docs/`/`llms.txt`, la section « Pointeurs » peut renvoyer au `README` / `CLAUDE.md`. Pour mettre en place une vraie doc structurée, utiliser `/project-setup`.

## Règles

1. **Mémoire, pas doc** — état courant et reprise, pas l'architecture stable (→ `docs/`).
2. **À jour** — mettre à jour après chaque edit ; supprimer ce qui est périmé. Une mémoire obsolète est pire qu'absente.
3. **Concision** — chaque ligne apporte de l'info utile à la reprise. Pas de remplissage.
4. **Factuel** — ne noter que du confirmé. Pas de spéculation.
5. **Pas de duplication** — si l'info existe dans `docs/`, `llms.txt` ou `CLAUDE.md`, y renvoyer plutôt que la recopier.
6. **Sections optionnelles** — omettre une section vide (ex: pas de gotchas).
