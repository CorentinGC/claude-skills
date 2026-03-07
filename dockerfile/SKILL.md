---
name: dockerfile
description: Bonnes pratiques Dockerfile — multi-stage builds, layer caching, non-root user, .dockerignore, health checks. Appliqué lors de la création ou modification de Dockerfiles.
---

# Skill : Dockerfile Best Practices

Ce skill s'applique automatiquement lors de la création ou modification de Dockerfiles et fichiers docker-compose.

## Règles à respecter

### 1. Multi-stage builds

Séparer le build de l'image finale pour réduire la taille.

```dockerfile
# HIGH — image unique avec les devDependencies
FROM node:20
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build
CMD ["node", "dist/index.js"]
# Résultat : image de ~1GB avec les sources, node_modules complets, etc.

# Correctif — multi-stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./
CMD ["node", "dist/index.js"]
```

Pour Next.js, utiliser le output `standalone` :

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/public ./public
CMD ["node", "server.js"]
```

---

### 2. Layer caching — ordre des instructions

Placer les instructions les moins changeantes en premier.

```dockerfile
# HIGH — cache invalidé à chaque changement de code
COPY . .
RUN npm ci

# Correctif — copier d'abord les fichiers de dépendances
COPY package*.json ./
RUN npm ci
COPY . .
```

Ordre recommandé :
1. Image de base
2. Outils système (`apt-get`, `apk add`)
3. Fichiers de dépendances (`package.json`, `pnpm-lock.yaml`)
4. Installation des dépendances
5. Code source
6. Build

---

### 3. Image de base légère

| Image | Taille | Usage |
|-------|--------|-------|
| `node:20` | ~1GB | Éviter en production |
| `node:20-slim` | ~200MB | Acceptable |
| `node:20-alpine` | ~130MB | Recommandé |
| `distroless` | ~30MB | Idéal pour la sécurité |

```dockerfile
# Préférer alpine
FROM node:20-alpine

# Si besoin de libc (certaines dépendances natives)
FROM node:20-slim
```

---

### 4. Utilisateur non-root

```dockerfile
# HIGH — exécution en root par défaut
FROM node:20-alpine
WORKDIR /app
COPY . .
CMD ["node", "index.js"]

# Correctif — utilisateur dédié
FROM node:20-alpine
WORKDIR /app
COPY . .
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
RUN chown -R appuser:appgroup /app
USER appuser
CMD ["node", "index.js"]

# Note : les images node:* incluent déjà un user 'node'
USER node
```

---

### 5. .dockerignore

Toujours créer un `.dockerignore` pour exclure les fichiers inutiles du context :

```
node_modules
.git
.gitignore
.env*
*.md
.next
dist
coverage
.turbo
.cache
Dockerfile*
docker-compose*
.dockerignore
```

---

### 6. npm ci vs npm install

```dockerfile
# MEDIUM — npm install peut modifier le lockfile
RUN npm install

# Correctif — npm ci respecte le lockfile exactement
RUN npm ci

# Production only
RUN npm ci --omit=dev
```

---

### 7. Health checks

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/api/health || exit 1
```

Pour les images Alpine (pas de curl par défaut), utiliser `wget` ou installer `curl`.

Implémenter un endpoint `/api/health` minimal :

```typescript
// GET /api/health
export function GET() {
  return Response.json({ status: 'ok' });
}
```

---

### 8. Variables d'environnement

```dockerfile
# Définir les valeurs par défaut non sensibles
ENV NODE_ENV=production
ENV PORT=3000

# Ne JAMAIS mettre de secrets dans le Dockerfile
# HIGH — secret dans l'image
ENV DATABASE_URL=postgres://user:password@host/db

# Correctif — passer au runtime
# docker run -e DATABASE_URL=... ou via docker-compose
```

---

### 9. Réduire les layers

```dockerfile
# MEDIUM — 3 layers pour les dépendances système
RUN apk update
RUN apk add --no-cache curl
RUN apk add --no-cache openssl

# Correctif — une seule layer + nettoyage
RUN apk add --no-cache curl openssl
```

---

### 10. Signaux et arrêt graceful

```dockerfile
# Utiliser exec form (pas shell form) pour recevoir les signaux
# HIGH — shell form : node ne reçoit pas SIGTERM
CMD npm start

# Correctif — exec form
CMD ["node", "dist/index.js"]

# Ou avec dumb-init pour la gestion des signaux
RUN apk add --no-cache dumb-init
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "dist/index.js"]
```

---

### 11. Docker Compose

```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
      target: runner  # Utiliser le stage spécifique
    restart: unless-stopped
    env_file: .env
    ports:
      - "${PORT:-3000}:3000"
    depends_on:
      db:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "wget", "--spider", "http://localhost:3000/api/health"]
      interval: 30s
      timeout: 3s
      retries: 3

  db:
    image: postgres:16-alpine
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  pgdata:
```

---

## Application

Lors de la création ou modification d'un Dockerfile :

1. Vérifier le multi-stage build.
2. Vérifier l'ordre des layers pour le cache.
3. Vérifier l'image de base (alpine/slim).
4. Vérifier l'utilisateur non-root.
5. Vérifier la présence du `.dockerignore`.
6. Vérifier le health check.
7. Vérifier qu'aucun secret n'est dans l'image.
