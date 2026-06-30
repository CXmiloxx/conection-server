# Plantilla — repo raíz de orquestación

Maqueta lista para copiar: un **repo raíz** que orquesta varios servicios con Docker Compose. Cada servicio es un **submódulo de Git** = un repositorio independiente de la organización.

> Esto es una **plantilla con marcadores** (`org`, `SERVICIO-API-A`, `WEB-ADMIN`, `DATABASE`). Sustitúyelos por los valores reales de tu proyecto. Para la teoría, ver:
> - [Submódulos](../../git/submodulos.md) — cómo se conectan los repos.
> - [Dockerfiles](../dockerfiles.md) — los `Dockerfile` / `Dockerfile.dev` que esta plantilla construye.
> - [Docker Compose](../docker-compose.md) — explicación de cada campo.
> - [Arquitectura](../../arquitectura/orquestacion-submodulos-docker.md) — visión global.

---

## Qué incluye

| Archivo | Para qué |
|---|---|
| `docker-compose.yml` | Orquestación de **producción** (multi-stage, límites de recursos). |
| `docker-compose.dev.yml` | **Desarrollo**: bind mounts, hot-reload, volúmenes para `node_modules` y artefactos. |
| `.env.example` | Plantilla de puertos del raíz → copiar a `.env`. |

La maqueta trae 3 servicios de ejemplo: 2 backends (`SERVICIO-API-A`, `SERVICIO-API-B`) y 1 frontend (`WEB-ADMIN`). Los backends comparten un submódulo anidado de schema (`DATABASE`) que genera artefactos al arrancar (`pnpm db:generate`).

---

## Estructura del proyecto resultante

```
org-RAIZ/                       ← este repo (orquestador, sin código de app)
├── SERVICIO-API-A/             ← submódulo: repo independiente de la org
│   └── DATABASE/               ← submódulo anidado: schema compartido
├── SERVICIO-API-B/             ← submódulo
│   └── DATABASE/               ← mismo repo de schema
├── WEB-ADMIN/                  ← submódulo: frontend
├── docker-compose.yml          ← producción
├── docker-compose.dev.yml      ← desarrollo
├── .env.example                ← plantilla (versionada)
├── .env                        ← puertos reales (NO versionado)
└── README.md
```

Cada submódulo lleva sus propios `Dockerfile`, `Dockerfile.dev`, `.dockerignore` y `.env`.

---

## Cómo adaptar la plantilla

1. **Prefijo de organización:** reemplaza `org` por el prefijo de tu organización en minúsculas (afecta nombres de contenedor, imágenes, red y volúmenes). Usa solo minúsculas, dígitos y `_`/`-`: evita `<`, `>` y espacios o romperás el esquema YAML.
2. **Servicios:** renombra `api_servicio_a` / `api_servicio_b` / `web_admin` y sus `context: ./SERVICIO-...` por los nombres reales de tus submódulos. Duplica o elimina bloques según cuántos servicios tengas.
3. **Puertos:** ajusta `API_A_PORT`, `API_B_PORT`, `ADMIN_PORT` en `.env.example` y los puertos internos en cada servicio.
4. **Schema compartido:** si NO usas un submódulo de DB, borra las líneas de `*_db_generated`, el volumen correspondiente y cambia el `command` a `pnpm start:dev`.
5. **Recursos:** ajusta `cpus`/`memory` en `deploy.resources` según el peso de cada servicio.

---

## Arranque

**Sincronizar submódulos** (tras clonar el raíz):

```bash
git submodule update --init --recursive
# o, al clonar:  git clone --recurse-submodules <url>
```

**Configurar puertos:**

```bash
cp .env.example .env
# además, crear el .env de cada submódulo a partir de su .env.example
```

**Desarrollo:**

```bash
docker compose -f docker-compose.dev.yml up
```

**Producción:**

```bash
docker compose -f docker-compose.yml up -d --build
```

**Logs de un servicio:**

```bash
docker compose -f docker-compose.dev.yml logs -f api_servicio_a
```

**Detener / reset:**

```bash
docker compose -f docker-compose.dev.yml down       # detener
docker compose -f docker-compose.dev.yml down -v    # + borrar volúmenes (reset)
```

---

## Comunicación entre servicios

Dentro de la red (`org_*_network`) los servicios se resuelven **por nombre de servicio**, no por `localhost`. Desde el frontend, la API A está en:

```text
http://api_servicio_a:3000
```

Más detalle en [docker-compose §4](../docker-compose.md).
