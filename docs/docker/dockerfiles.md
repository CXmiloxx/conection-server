# Dockerfiles por tipo de servicio

Plantillas de Dockerfile para los servicios de un proyecto orquestado con [submódulos](../git/submodulos.md) y [Docker Compose](./docker-compose.md). Cada submódulo lleva **sus propios** Dockerfiles.

| Archivo | Entorno | Estrategia |
|---|---|---|
| `Dockerfile` | Producción | Multi-stage: build + runtime mínimo |
| `Dockerfile.dev` | Desarrollo | Single-stage con hot-reload |
| `.dockerignore` | Ambos | Excluye `node_modules`, `.git`, builds, `.env` |

---

## 1. ¿Por qué multi-stage en producción?

Una imagen multi-stage usa una etapa para **compilar** (con todas las devDependencies, TypeScript, etc.) y otra **final limpia** que solo copia los artefactos compilados. Resultado: imágenes mucho más pequeñas, sin código fuente ni herramientas de build, con menor superficie de ataque.

En desarrollo no interesa eso: interesa **hot-reload**. Por eso dev es single-stage, monta el código con bind mount (ver [docker-compose](./docker-compose.md)) y ejecuta el servidor en modo watch.

---

## 2. `.dockerignore` (imprescindible)

Sin esto, `COPY . .` mete `node_modules` del host (compilados para otra arquitectura), el `.git` y secretos. Crea un `.dockerignore` en cada submódulo:

```gitignore
node_modules
dist
.next
.git
.gitignore
*.log
.env
.env.*
Dockerfile*
docker-compose*
```

---

## 3. Backend (Node.js / NestJS / Express)

### `Dockerfile` — producción (multi-stage)

Stage 1 compila. Stage 2 es la imagen final limpia, sin TypeScript ni devDependencies.

```dockerfile
FROM node:22-alpine AS builder

WORKDIR /app

RUN npm install -g pnpm@10.33.3

COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build

# ─────────────────────────────────────
FROM node:22-alpine

WORKDIR /app

RUN npm install -g pnpm@10.33.3

COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile --prod

COPY --from=builder /app/dist ./dist

EXPOSE <PUERTO>

CMD ["node", "dist/src/main.js"]
```

Notas:

- `--frozen-lockfile` falla si el lockfile no coincide con `package.json`: builds reproducibles.
- Copiar `package.json` + lock **antes** del resto aprovecha la caché de capas: si no cambian las dependencias, Docker no reinstala.
- `--prod` en la etapa final omite devDependencies.

Si el servicio usa un submódulo de schema/recursos generados (ej. Prisma), genera los artefactos **antes** del build:

```dockerfile
# Después de COPY . .
RUN pnpm db:generate   # script que genera artefactos del submódulo
RUN pnpm build
```

### `Dockerfile.dev` — desarrollo (single-stage, hot-reload)

```dockerfile
FROM node:22-alpine

WORKDIR /app

RUN npm install -g pnpm@10.33.3

ENV PNPM_HOME="/root/.local/share/pnpm"
ENV PATH="$PNPM_HOME:$PATH"

COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile

COPY . .

EXPOSE <PUERTO>

CMD ["pnpm", "start:dev"]
```

Si necesita generar artefactos del submódulo al arrancar (ej. cliente Prisma):

```dockerfile
CMD ["sh", "-c", "pnpm db:generate && pnpm start:dev"]
```

---

## 4. Frontend (Next.js / React)

### `Dockerfile` — producción (multi-stage)

```dockerfile
FROM node:22-alpine AS builder

WORKDIR /app

RUN npm install -g pnpm@10.33.3

COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build

# ─────────────────────────────────────
FROM node:22-alpine

WORKDIR /app

RUN npm install -g pnpm@10.33.3

ENV NODE_ENV=production
ENV PORT=<PUERTO>

COPY package.json pnpm-lock.yaml ./
RUN pnpm install --prod --frozen-lockfile

COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public
COPY --from=builder /app/next.config.mjs ./

EXPOSE <PUERTO>

CMD ["pnpm", "start"]
```

> En Next.js, si usas `output: "standalone"` en `next.config.mjs`, la imagen final puede ser aún más pequeña copiando solo `.next/standalone` y `.next/static`.

### `Dockerfile.dev` — desarrollo (single-stage, hot-reload)

```dockerfile
FROM node:22-alpine

WORKDIR /app

RUN npm install -g pnpm@10.33.3

ENV PNPM_HOME="/root/.local/share/pnpm"
ENV PATH="$PNPM_HOME:$PATH"
ENV PORT=<PUERTO>
ENV NODE_ENV=development

COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile

COPY . .

EXPOSE <PUERTO>

CMD ["pnpm", "dev"]
```

---

## 5. Adaptar a npm o yarn

Las plantillas usan **pnpm**. Para otros gestores, sustituye:

| pnpm | npm | yarn |
|---|---|---|
| `pnpm install --frozen-lockfile` | `npm ci` | `yarn install --frozen-lockfile` |
| `pnpm install --prod` | `npm ci --omit=dev` | `yarn install --production` |
| `pnpm build` | `npm run build` | `yarn build` |
| Lockfile: `pnpm-lock.yaml` | `package-lock.json` | `yarn.lock` |

---

## 6. Buenas prácticas

- **Fija versiones:** `node:22-alpine` y `pnpm@10.33.3` exactos → builds reproducibles.
- **Orden de capas:** dependencias antes del código fuente, para cachear instalaciones.
- **`alpine`** reduce el tamaño; si una dependencia nativa falla en Alpine, usa `node:22-slim`.
- **No copies `.env`** a la imagen: las variables se inyectan en runtime (ver [docker-compose](./docker-compose.md)).
- **Usuario no-root** en producción para reducir riesgo:
  ```dockerfile
  USER node
  ```
- **Healthcheck** opcional para que Docker conozca el estado real del servicio:
  ```dockerfile
  HEALTHCHECK --interval=30s --timeout=3s \
    CMD wget -qO- http://localhost:<PUERTO>/health || exit 1
  ```
