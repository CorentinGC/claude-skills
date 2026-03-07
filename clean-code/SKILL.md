---
name: clean-code
description: Applique les principes de clean code lors de l'écriture ou la modification de code — DRY, SRP, < 500 LOC, naming explicite, early returns, pas de dead code, colocation.
---

# Skill : Clean Code

Ce skill est appliqué automatiquement lors de l'écriture ou la modification de code.

## Principes à respecter

### 1. DRY — Don't Repeat Yourself

- Ne jamais dupliquer de logique. Si un bloc de code apparaît 2+ fois, l'extraire dans une fonction, un hook, ou un composant.
- Chercher l'existant avant de créer : vérifier si un utilitaire, hook ou composant similaire existe déjà dans le projet.
- Les constantes partagées doivent vivre dans un fichier `constants.ts` dédié.

### 2. Single Responsibility Principle (SRP)

- 1 fichier = 1 responsabilité claire.
- Un composant React ne doit pas mélanger : logique métier, appels API, et rendu UI.
- Extraire la logique métier dans des hooks ou des utilitaires.
- Extraire les types dans des fichiers `.types.ts` quand ils sont partagés.

### 3. Limite de taille — max 500 LOC

- **Warning** à 400 LOC → proposer un découpage.
- **Erreur** à 500+ LOC → découper obligatoirement.
- Stratégies de découpage :
  - Sous-composants pour les parties de rendu
  - Hooks custom pour la logique réutilisable
  - Fichiers utilitaires pour les fonctions pures
  - Fichiers de constantes et de types séparés

### 4. Naming explicite

- Pas de noms vagues : ~~`data`~~, ~~`info`~~, ~~`handleClick`~~, ~~`temp`~~, ~~`utils`~~.
- Les fonctions décrivent ce qu'elles font : `formatSid`, `buildClassifierSamples`, `calculateTotalPrice`.
- Les booléens commencent par `is`, `has`, `can`, `should` : `isLoading`, `hasPermission`.
- Les handlers décrivent l'action : `handleDeleteUser`, `handleSubmitForm`.
- Les composants décrivent ce qu'ils affichent : `UserProfileCard`, `MissionFilterBar`.

### 5. Early returns — pas de nesting profond

- Maximum 3 niveaux d'indentation dans une fonction.
- Utiliser des guard clauses en début de fonction pour les cas d'erreur ou les cas limites.
- Préférer `if (!condition) return` plutôt que `if (condition) { ... long bloc ... }`.

```typescript
// ❌ Mauvais
function processUser(user: User) {
  if (user) {
    if (user.isActive) {
      if (user.hasPermission) {
        // logique...
      }
    }
  }
}

// ✅ Bon
function processUser(user: User) {
  if (!user) return;
  if (!user.isActive) return;
  if (!user.hasPermission) return;
  // logique...
}
```

### 6. Pas de dead code

- Ne jamais laisser de code commenté (« au cas où »). Git est là pour ça.
- Supprimer les variables, imports et fonctions inutilisés.
- Pas de `// TODO` sans issue ou ticket associé.

### 7. Colocation

- Les fichiers vivent au plus proche de là où ils sont utilisés.
- Un composant utilisé uniquement dans une page → dans le dossier `components/` de cette page.
- Un hook utilisé dans un seul module → dans le dossier de ce module.
- Seuls les éléments partagés entre plusieurs modules montent dans les dossiers globaux (`src/components/`, `src/hooks/`, etc.).

## Application

Ces règles s'appliquent à **tout code écrit ou modifié**. Lors de la revue ou l'écriture de code :

1. Vérifier que le code respecte ces principes.
2. Si une violation est détectée, la corriger directement ou la signaler.
3. Ne pas sur-ingénierer — ces règles servent la lisibilité, pas la complexité.
