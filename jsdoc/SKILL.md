---
name: jsdoc
description: Applique les conventions JSDoc Eden Solutions sur tout fichier créé ou modifié — header fichier avec @author, documentation des fonctions exportées avec @param et @returns.
---

# Skill : JSDoc — Conventions Eden Solutions

Ce skill s'applique automatiquement lors de la création ou modification de fichiers TypeScript/JavaScript.

## Header de fichier

Chaque fichier doit avoir un header JSDoc en première position :

```typescript
/**
 * Description courte du fichier (1 ligne).
 * @author Eden Solutions <contact@eden-solutions.pro>
 */
```

- La description est concise et en français.
- `@author` est toujours `Eden Solutions <contact@eden-solutions.pro>`.
- Pas de `@created`, `@copyright`, ou `@file` — rester minimaliste.

## Fonctions exportées

Toute fonction exportée (`export function`, `export const`) doit avoir un JSDoc :

```typescript
/**
 * Description de ce que fait la fonction (1-2 lignes).
 * @param nomParam - Description du paramètre.
 * @returns Description du retour.
 */
export const formatSid = (
  sid: number | string | null | undefined,
  fallback = '----',
): string => {
```

### Règles des @param

- Format : `@param nomParam - Description.` (pas de type entre `{}`, TypeScript le gère).
- Un `@param` par paramètre.
- Mentionner la valeur par défaut si elle existe : `@param fallback - Valeur de fallback (défaut: "----").`
- Omettre `@param` pour les paramètres évidents (ex: `children` dans un composant React).

### Règles des @returns

- Format : `@returns Description.`
- Omettre si le retour est évident (void, JSX d'un composant React).
- Inclure un exemple si le format est non trivial : `@returns Le SID formaté sur 4 chiffres (ex: 55 → "0055").`

## Composants React

Les composants React exportés ont un JSDoc simplifié :

```typescript
/**
 * Affiche le profil utilisateur avec ses informations et permissions.
 */
export default function UserProfile({ user }: UserProfileProps): ReactElement {
```

- Description uniquement, pas de `@param` pour les props (le type le fait).
- Pas de `@returns` (c'est toujours du JSX).

## Ce qui n'a PAS besoin de JSDoc

- Fonctions internes (non exportées) dont le nom est explicite.
- Fichiers de types purs (`.types.ts`) — les types se documentent eux-mêmes.
- Fichiers de constantes simples — un commentaire inline suffit.
- Les `index.ts` de re-export.
- Les fichiers de configuration (`next.config.ts`, `tailwind.config.ts`...).

## Constantes et objets exportés

Les constantes exportées significatives ont un JSDoc court :

```typescript
/**
 * Indicateurs de différences utilisés dans les extractions RDPC.
 */
export const DiffFlag = {
  NONE: "none",
  ADDED: "added",
  REMOVED: "removed",
} as const;
```

## Langue

- Les JSDoc sont rédigés en **français**, comme le reste du projet.
- Les noms de paramètres restent en anglais (convention code).

## Application

Lors de l'écriture ou la modification de code :

1. Ajouter le header fichier s'il est absent.
2. Documenter les fonctions exportées nouvelles ou modifiées.
3. Ne pas sur-documenter — le code lisible se suffit à lui-même.
4. Mettre à jour le JSDoc si la signature ou le comportement change.
