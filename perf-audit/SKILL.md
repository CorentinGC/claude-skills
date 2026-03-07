---
name: perf-audit
description: Audit de performance web — bundle size, lazy loading, memoization, images, Core Web Vitals, Server vs Client Components. À invoquer avec /perf-audit.
user_invocable: true
---

# Skill : Performance Audit

Audit de performance complet pour applications web JavaScript/TypeScript (Next.js, React, Node.js). Couvre le rendu, le réseau, le bundle, les images et les Core Web Vitals.

## Procédure d'audit

Exécuter chaque section dans l'ordre. Pour chaque problème trouvé, indiquer l'impact estimé (HIGH / MEDIUM / LOW) et proposer un correctif.

---

### 1. Bundle size et tree-shaking

- Vérifier les imports non tree-shakables (import de la lib entière au lieu du module)
- Rechercher les libs lourdes avec des alternatives légères

```typescript
// HIGH — import de toute la lib
import _ from 'lodash';
// Correctif
import debounce from 'lodash/debounce';

// HIGH — moment.js (330kb) → date-fns ou dayjs
import moment from 'moment';
// Correctif
import { format } from 'date-fns';
```

- Vérifier les dépendances dupliquées (`npm ls <pkg>`)
- Vérifier les polyfills inutiles pour les navigateurs ciblés
- Vérifier les fichiers JSON/data volumineux importés dans le bundle client

---

### 2. Code splitting et lazy loading

- Vérifier que les routes/pages sont lazy-loadées
- Rechercher les composants lourds chargés au premier rendu qui pourraient être différés
- Vérifier les modals, drawers, onglets non visibles au chargement initial

```typescript
// MEDIUM — composant lourd chargé immédiatement
import HeavyChart from './HeavyChart';

// Correctif
const HeavyChart = lazy(() => import('./HeavyChart'));
```

#### Next.js spécifique
- Vérifier l'utilisation de `next/dynamic` pour les composants client lourds
- Vérifier que les pages utilisent le streaming avec `loading.tsx`

---

### 3. Server Components vs Client Components (Next.js App Router)

- Vérifier les `'use client'` inutiles — un composant sans interactivité ni hooks doit rester Server Component
- Vérifier que `'use client'` est placé au plus bas dans l'arbre (pas sur un layout parent)
- Vérifier les données fetchées côté client qui pourraient l'être côté serveur
- Vérifier les composants qui importent des libs client-only (framer-motion, recharts) → isoler avec `next/dynamic`

```typescript
// HIGH — 'use client' trop haut, tout le sous-arbre devient client
// app/dashboard/layout.tsx
'use client'; // Pourquoi ? Juste pour un useState dans un enfant

// Correctif — déplacer 'use client' dans le composant qui en a besoin
```

---

### 4. Rendu et re-renders React

- Rechercher les re-renders inutiles :
  - Objets/arrays créés inline dans les props (`style={{}}`, `options={[]}`)
  - Fonctions recréées à chaque rendu sans `useCallback`
  - Context providers dont la value change à chaque rendu
- Vérifier les listes sans `key` stable (pas d'index comme key sur des listes dynamiques)
- Vérifier les composants coûteux sans `React.memo` quand le parent re-render souvent
- Vérifier les `useMemo` / `useCallback` manquants sur les calculs coûteux

```typescript
// MEDIUM — nouvelle référence à chaque rendu
<List items={data.filter(d => d.active)} />

// Correctif
const activeItems = useMemo(() => data.filter(d => d.active), [data]);
<List items={activeItems} />
```

- Ne PAS ajouter `useMemo`/`useCallback` partout — seulement quand il y a un problème mesurable ou un composant mémoïzé en enfant.

---

### 5. Images et médias

- Vérifier l'utilisation de `next/image` (Next.js) ou lazy loading natif
- Vérifier les images sans dimensions (`width`/`height`) → cause CLS
- Vérifier les images non optimisées (PNG pour des photos → WebP/AVIF)
- Vérifier les images au-dessus du fold sans `priority`
- Vérifier les images en dessous du fold sans `loading="lazy"`
- Vérifier les SVG inline volumineux qui pourraient être des fichiers externes
- Vérifier les vidéos sans `preload="none"` ou `preload="metadata"`

```typescript
// HIGH — image LCP sans priority
<Image src="/hero.jpg" alt="Hero" width={1200} height={600} />
// Correctif
<Image src="/hero.jpg" alt="Hero" width={1200} height={600} priority />
```

---

### 6. Réseau et data fetching

- Vérifier les waterfalls de requêtes (requêtes séquentielles qui pourraient être parallèles)
- Vérifier l'absence de cache sur les requêtes API répétées
- Vérifier les sur-fetching (récupérer 100 champs quand 5 suffisent)
- Vérifier les appels API dans des boucles (N+1 côté client)
- Vérifier la pagination — pas de chargement de listes entières

```typescript
// HIGH — waterfall
const user = await fetchUser(id);
const posts = await fetchPosts(user.id); // Attend le premier appel
const comments = await fetchComments(user.id); // Attend le premier appel

// Correctif — paralléliser les appels indépendants
const user = await fetchUser(id);
const [posts, comments] = await Promise.all([
  fetchPosts(user.id),
  fetchComments(user.id),
]);
```

#### Next.js spécifique
- Vérifier les `fetch()` sans options de cache dans les Server Components
- Vérifier les `revalidate` cohérents entre les routes qui partagent des données
- Vérifier l'utilisation de `generateStaticParams` pour les pages statiques

---

### 7. CSS et layout

- Vérifier les layout shifts (CLS) — éléments sans dimensions réservées
- Vérifier les animations coûteuses (propriétés qui triggent layout : `width`, `height`, `top`, `left`)
- Préférer `transform` et `opacity` pour les animations (GPU-accelerated)
- Vérifier les fonts — `font-display: swap` ou `next/font`
- Vérifier les CSS inutilisés (fichiers de styles importés mais non utilisés)

```css
/* MEDIUM — animation qui trigger layout */
.animate { transition: width 0.3s; }

/* Correctif — GPU-accelerated */
.animate { transition: transform 0.3s; }
```

---

### 8. API et backend (Node.js)

- Vérifier les opérations synchrones bloquantes (`fs.readFileSync`, `crypto` sync)
- Vérifier les requêtes DB non optimisées (voir skill `db-query-review`)
- Vérifier les réponses API volumineuses sans compression (gzip/brotli)
- Vérifier les middlewares appliqués globalement mais nécessaires sur peu de routes
- Vérifier le streaming pour les réponses volumineuses

---

### 9. Core Web Vitals checklist

| Métrique | Cible | Points de vérification |
|----------|-------|----------------------|
| **LCP** (Largest Contentful Paint) | < 2.5s | Image hero avec `priority`, fonts préchargées, pas de render-blocking JS |
| **FID/INP** (Interaction to Next Paint) | < 200ms | Pas de long tasks sur le main thread, event handlers rapides |
| **CLS** (Cumulative Layout Shift) | < 0.1 | Dimensions sur images/vidéos, fonts avec fallback, pas d'injection de contenu |
| **TTFB** (Time to First Byte) | < 800ms | Cache serveur, CDN, edge functions |
| **FCP** (First Contentful Paint) | < 1.8s | Critical CSS inline, pas de render-blocking resources |

---

## Format du rapport

```
## Rapport de performance — [Nom du projet]

### Résumé
- HIGH : X problèmes
- MEDIUM : Y problèmes
- LOW : Z problèmes

### Problèmes détectés

#### [HIGH] Titre du problème
- **Fichier** : `path/to/file.ts:42`
- **Catégorie** : Bundle / Rendu / Réseau / Images / CWV
- **Impact estimé** : ...
- **Correctif** : ...

### Recommandations générales
- ...
```

Corriger automatiquement les problèmes simples (imports, `priority`, `loading="lazy"`). Proposer un plan pour les optimisations structurelles (code splitting, architecture Server/Client Components).
