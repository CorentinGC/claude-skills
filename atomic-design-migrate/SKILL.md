---
name: atomic-design-migrate
description: Migre un projet existant vers le pattern Atomic Design — analyse les composants, propose un plan de migration, et restructure le code. A invoquer avec /atomic-design-migrate.
user_invocable: true
---

# Skill : Atomic Design — Migration

Migre un projet existant vers le pattern Atomic Design (atoms, molecules, organisms). Ce skill analyse les composants existants, propose un plan de migration, puis execute la restructuration.

## Procedure de migration

### Phase 1 — Inventaire

1. Scanner `src/components/` et les dossiers `components/` locaux dans `src/app/` (ou `src/pages/`).
2. Lister tous les composants trouves avec :
   - Nom du composant
   - Chemin actuel
   - Fichiers associes (styles, types, tests)
   - Imports internes (quels autres composants il utilise)
   - Nombre d'utilisations dans le projet (combien de fichiers l'importent)

### Phase 2 — Classification

Pour chaque composant, determiner son niveau :

**Atom** si :
- Rendu d'un seul element HTML semantique
- Pas de logique metier
- Props simples (variant, size, disabled, children)
- N'importe aucun autre composant du projet

**Molecule** si :
- Compose 2+ atoms
- Etat local simple (toggle, hover)
- Pas de fetch de donnees, pas de logique metier complexe

**Organism** si :
- Compose des molecules et/ou atoms en sections completes
- Peut contenir de la logique metier, du state management, ou du data fetching

**Local** (ne pas migrer dans le UI-kit) si :
- Utilise dans un seul module/page
- Contient de la logique metier specifique a un domaine

### Phase 3 — Plan de migration

Presenter un tableau recapitulatif au format :

```
| Composant       | Chemin actuel                  | Classification | Destination                              |
|-----------------|--------------------------------|----------------|------------------------------------------|
| Button          | src/components/Button.tsx      | Atom           | src/components/atoms/Button/index.tsx     |
| SearchBar       | src/app/search/SearchBar.tsx   | Molecule       | src/components/molecules/SearchBar/       |
| DataTable       | src/components/DataTable.tsx   | Organism       | src/components/organisms/DataTable/       |
| MissionCard     | src/app/missions/MissionCard   | Local          | (reste en place)                          |
```

**Demander confirmation a l'utilisateur avant de continuer.**

### Phase 4 — Execution

Pour chaque composant a migrer :

1. **Creer le dossier** selon la structure standard :
   ```
   ComponentName/
   ├── index.tsx
   ├── ComponentName.module.scss    (si styles existants)
   ├── ComponentName.types.ts       (si > 5 types)
   └── components/                  (si sous-composants)
   ```

2. **Deplacer le code** dans la nouvelle structure.

3. **Mettre a jour tous les imports** dans le projet entier.
   - Chercher tous les fichiers qui importaient l'ancien chemin.
   - Remplacer par le nouveau chemin.

4. **Verifier les regles d'import entre niveaux** :
   - Atoms : pas d'import d'autres composants du projet.
   - Molecules : importent uniquement des atoms.
   - Organisms : importent des molecules et/ou atoms.
   - Si une violation est detectee, la signaler et proposer un refactoring.

5. **Creer les barrel exports** si necessaire (`index.ts` dans atoms/, molecules/, organisms/).

### Phase 5 — Verification

1. Lancer `npx tsc --noEmit` pour verifier l'absence d'erreurs TypeScript.
2. Lancer le linter (`npm run lint` ou equivalent).
3. Verifier que `npm run build` passe sans erreur.
4. Lister les imports mis a jour pour review.

## Regles importantes

- **Ne jamais migrer sans confirmation** : toujours presenter le plan et attendre la validation.
- **Migrer par batch** : si le projet a beaucoup de composants, proposer de migrer par groupes (d'abord les atoms, puis molecules, puis organisms).
- **Garder les composants locaux en place** : ne pas deplacer un composant utilise dans un seul module.
- **Preserver l'historique git** : faire un commit par phase ou par batch pour faciliter le review et le rollback.
- **Ne pas casser les tests** : si des tests existent, mettre a jour leurs imports aussi.
