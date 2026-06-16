---
name: project-setup
description: Amorce un projet proprement — interview du projet, génère CLAUDE.md + AGENTS.md (code propre + règles agent : doc officielle d'abord, installateurs officiels + dernière version), /docs + llms.txt (doc tenue à jour), MEMORY.md (mémoire de travail), agents projet suggérés (model optimisé), config MCP (.mcp.json + chrome-devtools dédié), et impose la parallélisation des subagents (5-10). Invoquer avec /project-setup.
user-invocable: true
argument-hint: [chemin?]
---

# Skill : Project Setup

Amorce un projet (neuf ou existant) avec une config agent complète et cohérente. `$ARGUMENTS` = chemin du projet (optionnel, défaut = répertoire courant).

**Principe directeur** : ne rien demander qui peut être détecté, ne rien écraser sans accord, tenir la doc et la mémoire à jour ensuite.

## Vue d'ensemble

| Phase | Produit |
|-------|---------|
| 0 | Détection auto du projet (stack, domaines, existant) |
| 1 | Interview ciblé (uniquement le non-détectable) |
| 2 | `CLAUDE.md` + `AGENTS.md` (code propre + règles agent + parallélisation) |
| 3 | `docs/` + `llms.txt` (doc structurée, indexée) |
| 4 | `MEMORY.md` (mémoire de travail) |
| 5 | Agents projet `.claude/agents/` (model optimisé) |
| 6 | MCP `.mcp.json` (chrome-devtools dédié + suggestions) |
| 7 | Récap |

---

## Phase 0 — Détection (parallèle)

Lancer **plusieurs Explore en parallèle** (dogfooding de la parallélisation) pour cartographier vite :
- **Stack** : `package.json` (+ lockfile pnpm/npm/yarn/bun), `tsconfig.json`, `next.config.*`, `vite.config.*`, `prisma/schema.prisma`, `drizzle.config.*`, `foundry.toml`/`hardhat.config.*`, `Cargo.toml`, `pyproject.toml`, `go.mod`, `composer.json`.
- **Infra** : `Dockerfile`, `docker-compose.yml`, `vercel.json`, `.github/workflows/`.
- **Existant à NE PAS écraser** : `CLAUDE.md`, `MEMORY.md`, `docs/`, `llms.txt`, `.mcp.json`, `.claude/agents/`, `.beads/`, `AGENTS.md`.

En déduire : runtime, framework, langage, ORM/DB, auth, et les **domaines** actifs (front / backend-DB / Solidity-Web3 / DevOps). Présenter une synthèse en 3-5 lignes avant l'interview.

> Si `CLAUDE.md`, `docs/`, `MEMORY.md` ou `.mcp.json` existent déjà : passer en **mode merge** (proposer des ajouts, ne jamais remplacer un fichier sans confirmation explicite).

> **Projet vide** (aucun manifest détecté) : proposer de scaffolder via l'**installateur officiel à la dernière version** (ex. `npx create-next-app@latest`, `npm create vite@latest`, `forge init`, `cargo new`) **avant** de générer la config. Confirmer outil + version avec l'utilisateur (vérifier la version courante : `npm view <pkg> version`).

---

## Phase 1 — Interview (AskUserQuestion, ≤4 questions/appel)

Pré-remplir avec la détection. Ne poser que l'utile. Sujets à couvrir (regrouper en 1-2 appels) :

1. **Stack** — confirmer/compléter le détecté (runtime, framework, ORM/DB, auth, state, styling).
2. **Tests** — inclure des règles de test ? framework (Vitest/Jest/Playwright/Foundry/pytest) ? TDD ? seuil de coverage ?
3. **Conventions** — lint/format (ESLint/Prettier/Biome), convention de commit (Conventional Commits ?), branching, **langue doc/commentaires** (défaut **FR**, identifiants **EN**).
4. **Déploiement** — Vercel / Docker / autre / aucun.
5. **Beads** — activer le workflow `bd` (tracking de tâches) ? Si oui, brancher `beads-setup`.
6. **Domaines à activer** — confirmer les volets (a11y + atomic-design pour front ; db-query-review pour backend ; solidity-natspec + smart-contract-audit pour EVM).

---

## Phase 2 — Générer `CLAUDE.md` + `AGENTS.md`

Créer (ou proposer un merge) à la racine. `CLAUDE.md` inclut `AGENTS.md` via `@AGENTS.md`. Remplir les `{{placeholders}}` depuis détection + interview. Omettre les sections non pertinentes.

````markdown
# {{Nom du projet}} — Guide agent

@AGENTS.md

> Lis ce fichier, `AGENTS.md`, `MEMORY.md` et `llms.txt` **avant** toute tâche de code.

## Stack
{{Runtime}} · {{Framework}} · {{Langage}} · {{ORM/DB}} · {{Auth}} · {{Autres libs structurantes}}

## Commandes
| But | Commande |
|-----|----------|
| Install | `{{install}}` |
| Dev | `{{dev}}` |
| Build | `{{build}}` |
| Test | `{{test}}` |
| Lint/Format | `{{lint}}` |

## Mémoire & documentation (obligatoire)
- **Avant de coder** : lire `MEMORY.md` (état courant) + `llms.txt` (index de la doc).
- **Après chaque edit** : mettre à jour `MEMORY.md` (état, prochaines étapes, gotchas).
- **À chaque changement d'archi/pattern/module** : mettre à jour `docs/` et `llms.txt`.
- `MEMORY.md` = mémoire de travail courte. `docs/` = doc détaillée stable. Ne pas dupliquer : se référencer.

## Qualité de code
- **DRY / SRP** : pas de duplication ; 1 fichier = 1 responsabilité. Chercher l'existant avant de créer.
- **Taille** : warning à 400 LOC, découpage obligatoire à 500.
- **Naming explicite** : pas de `data`/`temp`/`handleClick` ; booléens `is/has/can/should`.
- **Early returns** : guard clauses, max 3 niveaux d'indentation.
- **Pas de dead code** : ni code commenté, ni imports/vars inutilisés, ni `TODO` sans ticket.
- **Colocation** : le code vit au plus proche de son usage ; seul le partagé monte en global.

## Conventions du projet
- **Tests** : {{stratégie tests — framework, ce qui doit être testé, coverage}}
- **Lint/Format** : {{outil}} — lancer `{{lint}}` avant commit.
- **Commits** : {{convention — ex: Conventional Commits}}.
- **Validation des inputs** : {{ex: Zod sur toute entrée externe}}.
- **Langue** : doc/commentaires en {{FR/EN}}, identifiants en EN.
{{- Règles de domaine : front (a11y, atomic-design) / backend (requêtes DB, transactions) / Solidity (NatSpec, checks-effects-interactions)}}

## Parallélisation des subagents (par défaut)
Pour tout travail décomposable, **lancer 5 à 10 subagents en parallèle** (une seule réponse, plusieurs appels) au lieu de séquentiel :
- **Exploration / recherche** : plusieurs `Explore` ciblés (par module/dossier/convention).
- **Édition multi-fichiers** : un agent par fichier/zone indépendante.
- **Revue** : agents par dimension (sécurité, perf, DB, qualité) en parallèle.
- **Optimiser le `model` par agent** : `haiku` pour recherche/doc/mécanique, `sonnet` pour code/review, `opus` pour archi/debug difficile/audit.
- Ne séquentialiser que les vraies dépendances. Agréger les résultats en un rapport.

## Agents projet
{{Liste des agents de .claude/agents/ + globaux pertinents (prisma-reviewer, security-reviewer), avec leur usage.}}

## MCP
- `chrome-devtools` : debug/inspection navigateur (données isolées dans `.mcp/chrome-devtools/`).
{{- Autres MCP activés.}}

## Beads (si activé)
- `bd ready --json` en début de tâche ; `bd update <id> --claim` avant de travailler ; `bd close <id>` à la fin. Remplace `TodoWrite`.

## Pointeurs
- `AGENTS.md` — règles agent · `llms.txt` — index de la doc · `docs/` — doc détaillée · `MEMORY.md` — mémoire de travail · `.mcp.json` — serveurs MCP.
````

Puis générer `AGENTS.md` — règles **destinées aux agents IA** (standard multi-outils : Claude Code, Cursor…), inclus par `CLAUDE.md` :

````markdown
# AGENTS.md — {{Nom du projet}}

Règles pour les agents IA. Inclus par `CLAUDE.md` (`@AGENTS.md`).

## Lire la doc officielle avant de coder
- Tes données d'entraînement peuvent être périmées — la doc officielle/locale fait foi.
- Avant toute tâche touchant une lib structurante ({{Framework}}, {{ORM}}, {{libs}}…), lire sa doc : `node_modules/<lib>/docs/` ou le `llms.txt` du package si présents, sinon la doc en ligne via le MCP `context7`.

## Outillage : installateurs officiels + dernière version
- **Privilégier les installateurs/scaffolders officiels** plutôt que d'écrire la structure à la main : `create-next-app`, `npm create vite@latest`, `create-t3-app`, `forge init`, `cargo new`, etc.
- **Toujours la dernière version stable** (ex. **Next.js 16** actuellement). Vérifier la version courante avant d'installer (`npm view <pkg> version`, doc officielle, ou `context7`) — ne pas se fier à la mémoire.
- Préférer une dépendance maintenue à une réimplémentation maison, sauf raison explicite documentée.

## Conventions non négociables
{{- Convention lib/framework (ex: imports via subpath ; Server Components par défaut ; pas de memo si React Compiler).}}
{{- Sécurité/permissions spécifiques (ex: gating systématique des Server Actions / routes).}}

## Workflow
- Lire `MEMORY.md` + `llms.txt` avant de coder ; mettre à jour `MEMORY.md` après chaque edit.
- Paralléliser le travail indépendant (5–10 subagents) — cf. `CLAUDE.md`.
- Jamais de commit/push sans validation explicite ; respecter lint/build/tests du projet.
````

---

## Phase 3 — Créer `docs/` + `llms.txt`

Créer un squelette **seedé** avec le détecté (ne pas laisser vide) :

```
docs/
├── architecture.md     # Vue d'ensemble technique, structure des dossiers, flux
├── setup.md            # Install, variables d'env (.env.example), commandes run
├── conventions.md      # Style, patterns, organisation du code
├── decisions/          # ADR — un fichier par décision technique notable
│   └── 0001-template.md
└── modules/            # Un fichier par domaine/module métier (au fil de l'eau)
```

`llms.txt` à la racine (format spec llms.txt — référencé dans `CLAUDE.md`) :

```markdown
# {{Nom du projet}}

> {{Résumé en une phrase : ce que fait le projet + stack principale.}}

## Documentation
- [Architecture](docs/architecture.md) : structure, flux, choix techniques
- [Setup](docs/setup.md) : install, env, commandes
- [Conventions](docs/conventions.md) : style et patterns
- [Décisions (ADR)](docs/decisions/) : historique des choix

## Référence agent
- [CLAUDE.md](CLAUDE.md) : règles de travail
- [MEMORY.md](MEMORY.md) : mémoire de travail (état courant)

## Modules
{{- [Nom du module](docs/modules/xxx.md) : rôle — à compléter au fil du code.}}
```

> Règle (déjà dans `CLAUDE.md`) : tenir `docs/` + `llms.txt` à jour à chaque changement d'archi, pattern ou module. Déléguer cette tâche à l'agent `doc-maintainer` (haiku) quand il existe.

---

## Phase 4 — Créer `MEMORY.md` (mémoire de travail)

À la racine, committable. **Rôle distinct de `docs/`** : pas de la doc, mais l'état pour reprendre vite. **Lu avant de coder, mis à jour après chaque edit.**

```markdown
# MEMORY — {{Nom du projet}}

> Mémoire de travail. À LIRE avant de coder, à METTRE À JOUR après chaque edit. Concis (< 150 lignes). Doc détaillée → `llms.txt` / `docs/`.

## État courant
- {{Sur quoi on travaille / dernier point d'arrêt.}}

## Prochaines étapes
- [ ] {{...}}

## Décisions récentes
- {{AAAA-MM-JJ}} — {{décision + raison courte.}}

## Pièges / gotchas
- {{Ce qui surprend, contourne, ou casse facilement.}}

## Pointeurs
- Archi : `docs/architecture.md` · Setup : `docs/setup.md` · Index : `llms.txt`
```

---

## Phase 5 — Suggérer & créer des agents projet

Proposer (AskUserQuestion) selon les domaines détectés, **chaque agent avec son `model` optimisé**. Créer uniquement ceux choisis dans `.claude/agents/`.

**Règle d'optimisation du model :**
| Type de tâche | model |
|---------------|-------|
| Recherche / exploration read-only | `haiku` |
| Génération / sync de doc | `haiku` |
| Scaffolding tests, refactor mécanique | `haiku`/`sonnet` |
| Implémentation feature, review de code | `sonnet` |
| Architecture, debug complexe, audit sécu/Solidity | `opus` |

**Catalogue suggéré :**
- **Tous projets** : `explorer` (haiku, read-only), `doc-maintainer` (haiku, sync docs/llms.txt), `test-writer` (sonnet).
- **Front/React** : `react-reviewer` (sonnet).
- **Backend/DB** : réutiliser le global `prisma-reviewer` ; `migration-helper` (sonnet) si migrations.
- **Solidity/Web3** : `solidity-auditor` (opus).
- **Réutiliser les globaux** (ne pas recréer) : `prisma-reviewer`, `security-reviewer` — juste les lister dans `CLAUDE.md`.

Template d'agent :

```markdown
---
name: {{nom}}
description: {{Quand l'invoquer — déclencheur clair.}}
model: {{haiku|sonnet|opus}}
tools: Read, Grep, Glob{{, Edit, Write, Bash}}
---

{{System prompt : rôle, ce qu'il vérifie/produit, format de sortie.}}
```

---

## Phase 6 — Config MCP (`.mcp.json` racine)

**chrome-devtools** (toujours proposé) avec un dossier dédié **dans** le projet. Créer/merger `.mcp.json` :

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": [
        "-y",
        "chrome-devtools-mcp@latest",
        "--isolated=false",
        "--user-data-dir=.mcp/chrome-devtools",
        "--log-file=.mcp/chrome-devtools/debug.log"
      ]
    }
  }
}
```

- Le chemin est **relatif à la racine du projet** (Claude Code lance les MCP projet depuis la racine). Si la résolution échoue, basculer en chemin absolu.
- Créer le dossier `.mcp/chrome-devtools/` et l'ajouter à `.gitignore` (données runtime ; **`.mcp.json` reste commité**) :

```gitignore
# MCP runtime data
.mcp/chrome-devtools/
```

**Autres MCP suggérés selon le projet** (demander lesquels activer, puis merger dans `.mcp.json` sans écraser l'existant) :
- `context7` — doc à jour des libs (utile partout).
- Postgres/Prisma MCP — si DB relationnelle.
- Playwright — si tests E2E (signaler le recouvrement avec chrome-devtools).
- GitHub — si workflow PR/issues via `gh`.
- Vercel (plugin) — si déploiement Vercel.

> Merge non destructif : si `.mcp.json` existe, ajouter les clés sous `mcpServers` sans toucher aux serveurs déjà présents.

---

## Phase 7 — Récap

Afficher un résumé : fichiers créés/mergés (`CLAUDE.md`, `AGENTS.md`, `docs/`, `llms.txt`, `MEMORY.md`, `.mcp.json`, agents), MCP et agents activés, et les prochaines actions (`/security-review`, `/code-review ultra`, `/db-query-review`, `bd ready` si beads). Rappeler de redémarrer la session pour charger les MCP.

## Rappels

- **Ne jamais écraser** un fichier existant sans confirmation — proposer un merge.
- **Détecter avant de demander** — l'interview ne porte que sur le non-détectable.
- **Cohérence mémoire** : `CLAUDE.md` impose lire `MEMORY.md` avant / le MAJ après chaque edit — c'est le cœur du dispositif.
- **Pas de commit/push** sans validation explicite de l'utilisateur.
