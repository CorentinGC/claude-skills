---
name: security-audit
description: Audit de sécurité complet pour applications web (Next.js, React, Node.js, API) — OWASP Top 10, XSS, CSRF, injection, auth, headers, secrets, dépendances. À invoquer avec /security-audit.
user_invocable: true
---

# Skill : Security Audit

Audit de sécurité complet pour applications web JavaScript/TypeScript. Couvre le code client, serveur, API, configuration et dépendances.

## Procédure d'audit

Exécuter chaque section dans l'ordre. Pour chaque problème trouvé, indiquer la sévérité (CRITICAL / HIGH / MEDIUM / LOW) et proposer un correctif.

---

### 1. Secrets et données sensibles

Rechercher dans tout le codebase :

- Clés API, tokens, mots de passe hardcodés dans le code source
- Fichiers `.env` commités dans git (`git log --all --diff-filter=A -- '*.env*'`)
- Secrets côté client (variables sans `NEXT_PUBLIC_` exposées, ou `NEXT_PUBLIC_` contenant des secrets)
- Clés privées, certificats, fichiers `.pem` / `.key` dans le repo
- URLs avec credentials inline (`postgres://user:pass@...`)
- Vérifier `.gitignore` : `.env*`, `*.pem`, `*.key`, `credentials.*` doivent être ignorés

```bash
# Patterns à rechercher
grep -rn "password\s*=\s*['\"]" --include="*.{ts,tsx,js,jsx}" .
grep -rn "api[_-]?key\s*=\s*['\"]" --include="*.{ts,tsx,js,jsx}" .
grep -rn "secret\s*=\s*['\"]" --include="*.{ts,tsx,js,jsx}" .
grep -rn "Bearer\s" --include="*.{ts,tsx,js,jsx}" .
grep -rn "SK_\|sk_\|pk_" --include="*.{ts,tsx,js,jsx}" .
```

---

### 2. Injection (SQL, NoSQL, Command, LDAP)

#### SQL / ORM Injection
- Rechercher les requêtes SQL brutes avec interpolation de strings
- Vérifier l'utilisation de paramètres préparés (Prisma, Drizzle, Knex)
- Attention aux `$queryRaw` / `$executeRaw` dans Prisma sans paramètres bindés
- Vérifier les clauses `WHERE` construites dynamiquement

```typescript
// CRITICAL — injection SQL
prisma.$queryRaw`SELECT * FROM users WHERE id = ${userId}` // OK — tagged template
prisma.$queryRawUnsafe(`SELECT * FROM users WHERE id = ${userId}`) // CRITICAL — injection
```

#### Command Injection
- Rechercher `exec()`, `execSync()`, `spawn()`, `execFile()` avec des inputs utilisateur
- Vérifier que les arguments sont échappés ou passés en tableau (pas en string)

#### NoSQL Injection
- Vérifier les requêtes MongoDB avec des objets non validés (`{ $gt: "" }`)
- S'assurer que les inputs sont typés/validés avant d'être passés aux requêtes

---

### 3. Cross-Site Scripting (XSS)

- Rechercher `dangerouslySetInnerHTML` — chaque usage doit être justifié et sanitizé (DOMPurify)
- Vérifier les `document.write()`, `innerHTML`, `outerHTML`
- Vérifier les URLs dynamiques dans `href`, `src`, `action` — risque de `javascript:` protocol
- Vérifier les paramètres d'URL injectés dans le DOM sans échappement
- Vérifier les SVG inline avec du contenu dynamique
- Vérifier les iframes avec `src` dynamique

```typescript
// HIGH — XSS potentiel
<a href={userInput}>Link</a> // Risque javascript: protocol
// Correctif
<a href={sanitizeUrl(userInput)}>Link</a>
```

---

### 4. Authentification et sessions

- Vérifier que les tokens JWT sont validés côté serveur (signature + expiration + issuer)
- Vérifier que les secrets JWT sont suffisamment forts (>= 256 bits)
- Vérifier que les cookies de session ont les flags : `HttpOnly`, `Secure`, `SameSite=Strict|Lax`
- Vérifier l'absence de stockage de tokens dans `localStorage` (préférer `httpOnly` cookies)
- Vérifier la gestion de l'expiration et du refresh des tokens
- Vérifier les routes protégées — middleware auth sur toutes les routes sensibles
- Vérifier la logique de rôles/permissions (RBAC/ABAC) — pas de vérification côté client uniquement
- Vérifier la protection contre le brute force (rate limiting sur login/register/reset)

#### Next.js spécifique
- Vérifier le middleware `middleware.ts` — couvre-t-il toutes les routes protégées ?
- Vérifier les Server Actions — sont-elles protégées par auth ?
- Vérifier les Route Handlers (`app/api/`) — auth vérifiée sur chaque route ?

---

### 5. Cross-Site Request Forgery (CSRF)

- Vérifier la présence de tokens CSRF sur les formulaires et mutations
- Vérifier le header `SameSite` sur les cookies
- Vérifier que les actions sensibles (POST/PUT/DELETE) ne sont pas faisables via GET
- Vérifier la validation de l'origin/referer sur les endpoints sensibles

#### Next.js spécifique
- Les Server Actions incluent automatiquement une protection CSRF — vérifier qu'elle n'est pas désactivée
- Les Route Handlers API n'ont PAS de protection CSRF automatique — vérifier

---

### 6. Contrôle d'accès et autorisation

- Vérifier les IDOR (Insecure Direct Object Reference) : un utilisateur peut-il accéder aux ressources d'un autre en modifiant un ID dans l'URL/body ?
- Vérifier que chaque endpoint API vérifie les permissions de l'utilisateur authentifié
- Vérifier l'absence de endpoints admin accessibles sans vérification de rôle
- Vérifier les uploads de fichiers — type MIME, taille max, nom de fichier sanitizé
- Vérifier que les réponses API ne leakent pas de données sensibles (mots de passe hashés, tokens, emails d'autres users)

```typescript
// HIGH — IDOR
app.get('/api/users/:id', async (req, res) => {
  const user = await db.user.findUnique({ where: { id: req.params.id } });
  // Manque : vérifier que req.user.id === req.params.id ou que req.user est admin
});
```

---

### 7. Headers de sécurité HTTP

Vérifier la présence et la configuration correcte de ces headers :

| Header | Valeur recommandée |
|--------|-------------------|
| `Content-Security-Policy` | Politique restrictive, pas de `unsafe-inline` / `unsafe-eval` si possible |
| `X-Content-Type-Options` | `nosniff` |
| `X-Frame-Options` | `DENY` ou `SAMEORIGIN` |
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains; preload` |
| `Referrer-Policy` | `strict-origin-when-cross-origin` ou `no-referrer` |
| `Permissions-Policy` | Restreindre caméra, micro, géolocalisation selon besoin |
| `X-XSS-Protection` | `0` (désactivé — CSP le remplace) |

#### Next.js
- Vérifier `next.config.js` → `headers()` ou middleware
- Vérifier la config `images.remotePatterns` — pas de wildcard trop large

---

### 8. Validation des entrées

- Vérifier que TOUTES les entrées utilisateur sont validées côté serveur (Zod, Yup, joi, class-validator)
- La validation côté client seule n'est PAS suffisante
- Vérifier les types attendus : string vs number, longueur max, format (email, UUID)
- Vérifier les uploads : type MIME vérifié côté serveur (pas seulement l'extension)
- Vérifier les query params, path params, headers custom, body — tout est un input utilisateur
- Vérifier la protection contre le mass assignment (ne pas passer `req.body` directement à l'ORM)

```typescript
// HIGH — mass assignment
await prisma.user.update({ where: { id }, data: req.body }); // L'utilisateur peut modifier son rôle
// Correctif
const { name, email } = schema.parse(req.body);
await prisma.user.update({ where: { id }, data: { name, email } });
```

---

### 9. Dépendances et supply chain

```bash
# Vérifier les vulnérabilités connues
npm audit
# ou
pnpm audit
# ou
yarn audit
```

- Lister les dépendances avec des vulnérabilités critiques/hautes
- Vérifier les dépendances obsolètes majeures (`npm outdated`)
- Vérifier la présence d'un lockfile (`package-lock.json` / `pnpm-lock.yaml` / `yarn.lock`)
- Vérifier que le lockfile est commité
- Vérifier les scripts `postinstall` suspects dans les dépendances

---

### 10. Configuration et déploiement

- Vérifier que le mode debug/dev est désactivé en production
- Vérifier que les source maps ne sont pas exposées en production
- Vérifier les CORS — pas de `origin: '*'` sur des endpoints authentifiés
- Vérifier le rate limiting sur les API publiques
- Vérifier les timeouts et limites de taille de body sur les endpoints
- Vérifier la config HTTPS — redirection HTTP -> HTTPS
- Vérifier les variables d'environnement requises en production (pas de fallback dangereux)

```typescript
// MEDIUM — CORS trop permissif
cors({ origin: '*', credentials: true }) // CRITICAL — ne jamais combiner wildcard + credentials
cors({ origin: '*' }) // MEDIUM — acceptable uniquement sur API publique non authentifiée
```

---

### 11. Logging et monitoring

- Vérifier que les erreurs sont loggées (pas de `catch` vides)
- Vérifier que les logs ne contiennent pas de données sensibles (passwords, tokens, PII)
- Vérifier la présence de rate limiting logs (tentatives de brute force)
- Vérifier que les erreurs renvoyées au client ne leakent pas de stack traces en production

```typescript
// HIGH — leak d'information
app.use((err, req, res, next) => {
  res.status(500).json({ error: err.message, stack: err.stack }); // CRITICAL en prod
});
```

---

### 12. React / Client-side spécifique

- Vérifier que les données sensibles ne sont pas dans le state client (Redux/Zustand store visible dans les devtools)
- Vérifier les `useEffect` qui font des appels API sans cleanup (race conditions, memory leaks)
- Vérifier les redirections côté client basées sur des paramètres URL (open redirect)
- Vérifier les web workers et service workers — scope et permissions
- Vérifier le Content Security Policy pour les scripts inline

---

## Format du rapport

Produire un rapport structuré :

```
## Rapport de sécurité — [Nom du projet]

### Résumé
- CRITICAL : X
- HIGH : Y
- MEDIUM : Z
- LOW : W

### Problèmes détectés

#### [CRITICAL] Titre du problème
- **Fichier** : `path/to/file.ts:42`
- **Description** : ...
- **Impact** : ...
- **Correctif** : ...

(répéter pour chaque problème)

### Recommandations générales
- ...
```

Corriger automatiquement les problèmes CRITICAL et HIGH quand c'est possible sans casser le fonctionnement existant. Demander confirmation à l'utilisateur avant de modifier la logique métier.
