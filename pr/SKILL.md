---
name: pr
description: Crée une PR (head → base) si elle n'existe pas, sinon re-review la PR ouverte — en appliquant les règles du projet local (CLAUDE.md, AGENTS.md, llms.txt, docs, agents). Review qualité/sécurité/perf + plan de correction priorisé. Invoquer avec /pr.
user-invocable: true
argument-hint: [titre-optionnel]
---

# Skill : PR + review (règles du projet local)

Crée une pull request si aucune n'est ouverte, sinon re-review la PR existante. La review **s'appuie sur les conventions du projet courant** — pas de règles codées en dur ici.

## Étape 0 — Charger le contexte du projet

Avant toute action, lire les sources de règles présentes (ignorer celles qui manquent) :
- `CLAUDE.md` (+ fichiers `@inclus`, ex. `AGENTS.md`) — conventions, interdits, workflow, **flux PR éventuel**.
- `llms.txt` + `docs/` — patterns, architecture, doc à consulter selon le diff.
- `.claude/agents/` — agents de review du projet (sécurité, DB, conventions, permissions…) **et** toute orchestration de review décrite dans `CLAUDE.md` (ex. vagues d'agents).
- `package.json` (scripts) — commandes `lint` / `build` / `test` / e2e pour le test plan.
- `.beads/` ou `AGENTS.md` beads → utiliser `bd` pour le tracking ; sinon `TodoWrite`.

> **But** : la review doit refléter *ce projet-ci*. Construire les critères depuis ces fichiers. À défaut de `CLAUDE.md`, utiliser le checklist générique (Étape 2).

## Étape 1 — Détecter les branches et l'état de la PR

1. `git fetch origin --quiet`
2. **Branche base** (cible) : branche par défaut du repo.
   ```bash
   gh repo view --json defaultBranchRef -q .defaultBranchRef.name 2>/dev/null \
     || git symbolic-ref --short refs/remotes/origin/HEAD 2>/dev/null | sed 's@^origin/@@' \
     || echo main
   ```
3. **Branche head** (source) : la branche courante (`git rev-parse --abbrev-ref HEAD`).
   - Si le projet documente un flux fixe dans `CLAUDE.md` (ex. `staging → main`), le respecter.
   - Si head == base → stopper et demander à l'utilisateur quelle branche source utiliser.
4. **PR existante ?**
   ```bash
   gh pr list --base <BASE> --head <HEAD> --state open --json number,url,title,headRefOid
   ```

### Mode A — PR existante (re-review après fix)
- **Ne PAS recréer de PR.**
- Si `<HEAD>` local est en avance sur `origin/<HEAD>` → `git push origin <HEAD>` pour que la PR reflète le dernier fix.
- Annoncer : « PR #X déjà ouverte — re-review du diff à jour ».
- Aller directement à **Étape 2** sur `origin/<BASE>...origin/<HEAD>` (version remote). **Skipper** titre/body et `gh pr create`.

### Mode B — Pas de PR → création
1. Working tree propre ? `git status --porcelain` — si sale, stopper et demander (commit/stash/ignore).
2. Inspecter le diff :
   ```bash
   git log --oneline origin/<BASE>..<HEAD>
   git diff --stat origin/<BASE>...<HEAD>
   ```
3. Si `<HEAD>` local en avance sur `origin/<HEAD>` → `git push origin <HEAD>`.
4. Créer la PR :
   - Lire **tous** les commits du diff (pas seulement le dernier).
   - Titre court (< 70 car) basé sur le thème dominant ; si `$ARGUMENTS` fourni → l'utiliser.
   - Body :
     ```markdown
     ## Summary
     <changements majeurs groupés par module>

     ## Commits inclus
     <liste oneline>

     ## Test plan
     <cases dérivées des scripts du projet : lint, build, tests unitaires, e2e si présents>
     ```
   - `gh pr create --base <BASE> --head <HEAD> --title "..." --body "$(cat <<'EOF' … EOF)"`
5. Afficher l'URL de la PR.

## Étape 2 — Review automatique (parallèle, règles du projet)

Diff complet : `git diff origin/<BASE>...origin/<HEAD>`.

**Stratégie de parallélisation** (privilégier le parallèle, 1 seul message multi-`Agent`) :
- **Si le projet définit des agents de review** (`.claude/agents/` + orchestration dans `CLAUDE.md`) → les lancer **en parallèle** (sécurité, DB, conventions, permissions, perf…), en leur passant le diff + le contexte.
- **Sinon** → lancer **5 à 10 sous-agents génériques** (`Explore`/`general-purpose`) en parallèle : un par **axe** (ci-dessous) et, si gros diff, un par **module** touché. Optimiser le `model` (sécurité/archi → opus ; conventions/perf → sonnet ; recherche → haiku).
- Réutiliser les agents globaux pertinents selon la stack (`prisma-reviewer`, `security-reviewer`) quand ils s'appliquent.
- Diff < ~500 lignes et pas d'agents projet → analyser en direct.

Les **3 axes** (critères tirés de `CLAUDE.md`/`docs/`, complétés par ce socle générique) :

### Axe A — Qualité / conventions
Appliquer les règles du projet (taille de fichier, interdits, aliases d'import, patterns imposés…). Socle générique : fichiers trop longs, `console.log`/`debugger`, code mort/commenté, naming vague, magic strings, duplication, inputs non validés.

### Axe B — Sécurité
Auth/permissions manquantes sur endpoints/actions, injection (SQL/raw/command), XSS (`dangerouslySetInnerHTML` non sanitizé), secrets en dur / exposés côté client, IDOR / ownership, validation des inputs, upload non validé, fuite d'info dans les logs. **+ règles de sécurité spécifiques du projet** (multi-tenant, soft-delete bypass, scopes de permission…).

### Axe C — Performance
N+1 / over-fetching (`select` absent), composant client inutile, `useEffect` pour état dérivé, `await` séquentiels parallélisables (`Promise.all`), images/assets non lazy, revalidation/cache trop large, dépendances lourdes côté client, index DB manquants. Respecter les contraintes projet (ex. React Compiler actif → ne pas suggérer de memo).

## Étape 3 — Rapport + plan de correction

```markdown
# Review PR #<num> — <HEAD> → <BASE>

**Diff** : X fichiers, +Y/-Z lignes, N commits

## 🔴 Bloquants (à corriger avant merge)
- [fichier:ligne](chemin#Lxx) Description + fix suggéré

## 🟠 Recommandations fortes
- [fichier:ligne](chemin#Lxx) Description + impact

## 🟡 Améliorations optionnelles
- [fichier:ligne](chemin#Lxx) Description

## ✅ Points positifs
- Ce qui est bien fait

## 📋 Plan de correction
### Phase 1 — Bloquants (obligatoire)
1. **[Titre]** ([fichier:ligne]) — Problème · Correction · Effort S/M/L
### Phase 2 — Recommandations
### Phase 3 — Optimisations
```

## Étape 4 — Demander la suite

> Veux-tu que j'applique la Phase 1 du plan de correction ? (oui / non / sélection manuelle)

**Ne JAMAIS commiter ni pousser de correction sans validation explicite** (et respecter les règles du projet : pas de commit direct sur la base, hooks, etc.).

## Notes

- Référencer les fichiers en markdown cliquable : `[fichier.ts:42](src/fichier.ts#L42)`.
- Tracking : si `bd` est actif, créer une issue de review + une issue par bloquant Phase 1 (`bd create … -t bug -p 1`) ; sinon `TodoWrite`.
- Gros diff (> ~2000 lignes) → proposer de scinder la review par module.
- Ce skill **lit** les règles du projet ; il n'en impose aucune en dur. Adapter les axes à chaque repo.
