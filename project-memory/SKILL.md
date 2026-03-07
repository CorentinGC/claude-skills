---
name: project-memory
description: Maintient automatiquement un MEMORY.md a la racine de chaque projet (commitable) — stack, architecture, patterns, conventions, structure. Applique en debut de session et apres tout changement d'architecture ou de pattern.
---

# Skill : Project Memory

Ce skill s'applique automatiquement sur chaque projet. Il garantit qu'un fichier `MEMORY.md` existe et reste a jour dans le dossier memoire Claude du projet.

## Objectif

Fournir un contexte solide et synthetique pour que tout agent (ou developpeur) comprenne immediatement le projet sans scanner l'ensemble du codebase.

## Emplacement

Le fichier vit a la racine du projet, commite dans le repo :
```
./MEMORY.md
```

Cela permet a tous les contributeurs (humains et agents) de le consulter et le maintenir.

## Quand agir

### Creation (debut de session sur un nouveau projet)

Si le MEMORY.md du projet est vide ou inexistant :
1. Scanner les fichiers racine : `package.json`, `Cargo.toml`, `pyproject.toml`, `go.mod`, `composer.json`, `Gemfile`, `pom.xml`, `build.gradle`, etc.
2. Scanner la structure des dossiers (1-2 niveaux de profondeur)
3. Identifier les fichiers de config : `.env.example`, `docker-compose.yml`, `next.config.*`, `vite.config.*`, `tsconfig.json`, `hardhat.config.*`, `foundry.toml`, etc.
4. Lire les README, CLAUDE.md, ou fichiers de contexte existants
5. Generer le MEMORY.md selon le template ci-dessous

### Mise a jour (pendant le travail)

Mettre a jour le MEMORY.md apres :
- Ajout ou suppression d'une dependance majeure (framework, ORM, lib structurante)
- Changement de pattern ou convention (ex: migration de REST vers tRPC, ajout d'un state manager)
- Ajout d'un module ou domaine metier significatif
- Changement d'architecture (ex: monolith vers microservices, ajout d'un proxy, changement de DB)
- Ajout ou modification d'un systeme transversal (auth, permissions, caching, i18n, real-time)
- Migration technique (ex: changement de version majeure d'un framework)

**Ne PAS mettre a jour pour** : bugfixes, changements cosmetiques, ajout de composants individuels, travail en cours temporaire.

## Template MEMORY.md

Le fichier doit rester concis (< 200 lignes, les lignes au-dela sont tronquees dans le contexte). Privilegier la densite d'information.

```markdown
# MEMORY — [Nom du projet]

## Stack
[Runtime] · [Framework] · [Langage] · [ORM/DB] · [Auth] · [Autres libs structurantes]

## Structure
```
src/
├── [dossier]/          # [role — 1 ligne]
├── [dossier]/          # [role — 1 ligne]
└── ...
```

## Architecture
- [Type d'app : SPA, SSR, API, CLI, smart contract, monorepo, etc.]
- [Base de donnees : type, provider, ORM]
- [Auth : methode, provider]
- [API : REST, GraphQL, tRPC, RPC, Server Actions, etc.]
- [Real-time : WebSocket, SSE, Pusher, etc. — si applicable]
- [Deploiement : Vercel, Docker, AWS, etc. — si connu]

## Patterns et conventions
- [Convention 1 — ex: Server Components par defaut, "use client" uniquement si necessaire]
- [Convention 2 — ex: Validation Zod sur tous les inputs]
- [Convention 3 — ...]
- ...

## Modules / Domaines metier
| Module | Description | Fichiers cles |
|--------|-------------|---------------|
| [nom]  | [1 ligne]   | [chemins]     |

## Dependances cles
| Package | Role |
|---------|------|
| [nom]   | [1 ligne] |

## Decisions techniques notables
- [Decision 1 — ex: React Compiler actif, pas de memo/useMemo/useCallback]
- [Decision 2 — ex: soft delete avec deletedAt sur tous les modeles]
- ...

## Fichiers de reference
- [CLAUDE.md, .cursor/rules/, PROJECT_CONTEXT.md, etc. — chemins vers les docs internes]
```

## Regles

1. **Concision** — Chaque ligne doit apporter de l'information. Pas de phrases de remplissage.
2. **Factuel** — Ne documenter que ce qui est confirme dans le code. Ne pas speculer.
3. **Stable** — Ne documenter que les elements stables (architecture, patterns). Pas le travail en cours.
4. **Actionnable** — Chaque info doit aider l'agent a prendre de meilleures decisions.
5. **A jour** — Un MEMORY obsolete est pire qu'un MEMORY absent. Supprimer les infos obsoletes.
6. **Pas de duplication** — Si l'info existe deja dans un CLAUDE.md ou PROJECT_CONTEXT.md, y faire reference plutot que la copier. Le MEMORY est un complement, pas un doublon.
7. **Sections optionnelles** — Si une section n'est pas pertinente (ex: pas de DB, pas de real-time), l'omettre.
