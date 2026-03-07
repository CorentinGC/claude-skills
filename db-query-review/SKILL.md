---
name: db-query-review
description: Revue des requêtes DB — N+1, index manquants, select *, pagination, transactions, Prisma/Drizzle. À invoquer avec /db-query-review.
user_invocable: true
---

# Skill : Database Query Review

Revue des requêtes base de données dans les applications JavaScript/TypeScript. Couvre les ORMs (Prisma, Drizzle, Knex, TypeORM, Sequelize) et les requêtes SQL brutes.

## Procédure de revue

Examiner chaque section dans l'ordre. Pour chaque problème trouvé, indiquer la sévérité (HIGH / MEDIUM / LOW) et proposer un correctif.

---

### 1. Problème N+1

Rechercher les boucles qui exécutent une requête par itération.

```typescript
// HIGH — N+1 : 1 requête par user
const users = await prisma.user.findMany();
for (const user of users) {
  const posts = await prisma.post.findMany({ where: { authorId: user.id } });
}

// Correctif — une seule requête avec include
const users = await prisma.user.findMany({
  include: { posts: true },
});

// Ou batch query
const users = await prisma.user.findMany();
const posts = await prisma.post.findMany({
  where: { authorId: { in: users.map(u => u.id) } },
});
```

Patterns à rechercher :
- `for`/`forEach`/`map` contenant un `await` sur une requête DB
- `Promise.all` avec des requêtes unitaires dans un `.map()`
- Fonctions appelées en boucle qui font des requêtes en interne

---

### 2. Select * / Sur-fetching

- Vérifier les requêtes qui récupèrent tous les champs quand peu sont nécessaires
- Vérifier les `include` / `populate` qui chargent des relations entières inutilement

```typescript
// MEDIUM — récupère tous les champs dont le hash du mot de passe
const user = await prisma.user.findUnique({ where: { id } });

// Correctif — sélectionner uniquement les champs nécessaires
const user = await prisma.user.findUnique({
  where: { id },
  select: { id: true, name: true, email: true },
});
```

- Vérifier les réponses API qui renvoient l'objet DB brut (fuite de données)
- Vérifier les `include` imbriqués profonds (3+ niveaux)

---

### 3. Pagination

- Vérifier les `findMany()` sans `take` / `limit` → risque de charger toute la table
- Vérifier la pagination offset-based sur de grandes tables → préférer cursor-based
- Vérifier que le `count` total est récupéré efficacement

```typescript
// HIGH — charge potentiellement des millions de lignes
const allUsers = await prisma.user.findMany();

// Correctif — paginer
const users = await prisma.user.findMany({
  take: 20,
  skip: page * 20,
  orderBy: { createdAt: 'desc' },
});

// Mieux pour les grandes tables — cursor-based
const users = await prisma.user.findMany({
  take: 20,
  cursor: lastId ? { id: lastId } : undefined,
  orderBy: { id: 'asc' },
});
```

---

### 4. Index manquants

Vérifier les colonnes utilisées dans les clauses `WHERE`, `ORDER BY`, `JOIN` :

- Colonnes filtrées fréquemment sans index
- Colonnes de tri sur de grandes tables
- Clés étrangères sans index (fréquent en PostgreSQL)
- Index composites manquants pour les filtres multi-colonnes

```prisma
// Vérifier dans le schema Prisma
model Post {
  id        Int      @id @default(autoincrement())
  authorId  Int      // FK — vérifier qu'un index existe
  status    String   // Filtré souvent ? → ajouter @@index
  createdAt DateTime // Trié souvent ? → ajouter @@index

  @@index([authorId])
  @@index([status, createdAt])
}
```

Lister les requêtes et vérifier la correspondance avec les index du schéma.

---

### 5. Transactions

- Vérifier les opérations multi-tables sans transaction (risque d'incohérence)
- Vérifier les transactions trop larges (verrouillent longtemps)
- Vérifier les transactions imbriquées non supportées

```typescript
// HIGH — incohérence si la 2ème requête échoue
await prisma.order.create({ data: orderData });
await prisma.inventory.update({ where: { id: itemId }, data: { stock: { decrement: 1 } } });

// Correctif — transaction
await prisma.$transaction([
  prisma.order.create({ data: orderData }),
  prisma.inventory.update({ where: { id: itemId }, data: { stock: { decrement: 1 } } }),
]);
```

---

### 6. Requêtes dans les boucles de rendu (React)

- Vérifier les requêtes dans les composants Server qui pourraient être déduplicées
- Vérifier les `useEffect` qui refetchent à chaque re-render (deps manquantes/instables)
- Vérifier les requêtes identiques appelées par plusieurs composants sur la même page

```typescript
// MEDIUM — requête à chaque re-render
useEffect(() => {
  fetchUser(userId);
}); // Pas de deps → exécuté à chaque rendu

// Correctif
useEffect(() => {
  fetchUser(userId);
}, [userId]);
```

---

### 7. Requêtes brutes et SQL

- Vérifier les requêtes `$queryRaw` / `$executeRaw` — sont-elles nécessaires ou l'ORM suffit ?
- Vérifier les paramètres bindés (pas d'interpolation de string)
- Vérifier les types de retour (`$queryRaw` retourne `unknown[]` — typer correctement)

---

### 8. Migrations et schéma

- Vérifier les colonnes nullable sans valeur par défaut sur les nouvelles colonnes
- Vérifier les migrations destructives (drop column, rename) sans plan de rollback
- Vérifier les types de colonnes appropriés (TEXT vs VARCHAR, INT vs BIGINT)
- Vérifier les contraintes d'unicité manquantes (email, slug)
- Vérifier les cascades de suppression (`onDelete`) — bien configurées ?

---

### 9. Performance patterns

- `findFirst` vs `findUnique` — utiliser `findUnique` quand on cherche par clé unique
- `count()` vs `findMany().length` — ne jamais charger des données juste pour compter
- `upsert` vs `findFirst` + `create/update` — réduire le nombre de requêtes
- `createMany` / `updateMany` vs boucle de `create` / `update`
- Vérifier les `distinct` sur de grandes tables sans index

---

## Format du rapport

```
## Revue DB — [Nom du projet]

### Résumé
- HIGH : X problèmes
- MEDIUM : Y problèmes
- LOW : Z problèmes

### Problèmes détectés

#### [HIGH] Titre du problème
- **Fichier** : `path/to/file.ts:42`
- **Requête concernée** : ...
- **Impact** : ...
- **Correctif** : ...

### Recommandations
- Index à ajouter : ...
- Migrations suggérées : ...
```

Corriger automatiquement les N+1, sur-fetching et paginations manquantes quand c'est possible. Demander confirmation avant de modifier le schéma ou les migrations.
