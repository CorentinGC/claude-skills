---
name: atomic-design
description: Applique le pattern Atomic Design pour structurer les composants UI — atoms, molecules, organisms. À utiliser lors de la création de composants UI ou d'un UI-kit.
---

# Skill : Atomic Design — UI-Kit structuré

Ce skill s'applique automatiquement lors de la création ou modification de composants UI.

## Structure Atomic Design

Les composants UI suivent une hiérarchie en 3 niveaux :

```
src/components/
├── atoms/           # Éléments UI indivisibles
│   ├── Button/
│   ├── Badge/
│   ├── Input/
│   ├── Avatar/
│   └── Spinner/
├── molecules/       # Combinaisons d'atoms
│   ├── SearchBar/        # Input + Button
│   ├── UserBadge/        # Avatar + Badge + Text
│   ├── FormField/        # Label + Input + ErrorMessage
│   └── DropdownMenu/     # Button + List
├── organisms/       # Sections autonomes (composées de molecules/atoms)
│   ├── DataTable/
│   ├── Navbar/
│   ├── Sidebar/
│   └── FilterPanel/
```

## Règles de classification

### Atoms — Composants de base indivisibles

- Un seul élément HTML sémantique (button, input, span, img...).
- Pas de logique métier.
- Props simples : `variant`, `size`, `disabled`, `children`.
- Exemples : `Button`, `Badge`, `Input`, `Text`, `Icon`, `Avatar`, `Chip`.

### Molecules — Combinaisons d'atoms

- Composent 2+ atoms pour former un élément fonctionnel.
- Peuvent avoir un état local simple (toggle, hover).
- Pas de fetch de données, pas de logique métier complexe.
- Exemples : `SearchBar` (Input + Button), `FormField` (Label + Input + Error), `UserCard` (Avatar + Text + Badge).

### Organisms — Sections autonomes

- Composent des molecules et/ou atoms en sections complètes.
- Peuvent contenir de la logique métier et du state management.
- Peuvent fetcher des données.
- Exemples : `DataTable`, `Navbar`, `Sidebar`, `FilterPanel`, `CommentSection`.

## Règles de structure par composant

Chaque composant suit cette structure :

```
ComponentName/
├── index.tsx                    # Export principal
├── ComponentName.module.scss    # Styles (si CSS Modules)
├── ComponentName.types.ts       # Types (si > 5 types/interfaces)
└── components/                  # Sous-composants internes (si nécessaire)
```

## Règles d'import

- Les atoms n'importent **jamais** d'autres composants du projet (seulement des librairies).
- Les molecules importent uniquement des **atoms**.
- Les organisms importent des **molecules** et/ou des **atoms**.
- Les pages importent des **organisms**, **molecules**, et **atoms**.
- **Pas d'import circulaire** entre niveaux.

## Quand créer un composant partagé

Avant de créer un composant dans `src/components/` :

1. Est-il utilisé dans **2+ endroits** ? Si non → il reste local au module.
2. Est-il **générique** (pas de logique métier spécifique) ? Si non → il reste dans le module métier.
3. À quel niveau appartient-il ? Atom / Molecule / Organism ?

## Composants locaux vs partagés

```
src/
├── components/          # UI-Kit partagé (Atomic Design)
│   ├── atoms/
│   ├── molecules/
│   └── organisms/
├── app/
│   └── (main)/
│       └── missions/
│           └── components/  # Composants locaux à ce module
│               ├── MissionCard.tsx
│               └── MissionFilters.tsx
```

- **Local** : utilisé uniquement dans un module → `module/components/`
- **Partagé** : utilisé dans 2+ modules → `src/components/{atoms|molecules|organisms}/`

## Application

Lors de la création d'un composant UI :

1. Déterminer son niveau (atom/molecule/organism).
2. Vérifier s'il existe déjà dans le UI-kit.
3. Le placer au bon endroit (local vs partagé).
4. Respecter la structure de dossier standard.
5. Respecter les règles d'import entre niveaux.
