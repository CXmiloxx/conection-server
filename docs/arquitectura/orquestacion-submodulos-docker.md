# Arquitectura: orquestación de servicios con submódulos + Docker

Visión general del patrón completo: un **repositorio raíz** que orquesta múltiples **servicios independientes** mediante [git submodules](../git/submodulos.md) y [Docker Compose](../docker/docker-compose.md). Este documento es el mapa; los detalles de cada pieza están en sus guías enlazadas.

---

## 1. El patrón en una imagen

```
ORG-RAIZ/                        ← repo orquestador (sin código de app)
│
├── BACKEND/                     ← submódulo (repo independiente)
│   └── SCHEMA-COMPARTIDO/       ← submódulo anidado (recurso compartido)
├── FRONTEND/                    ← submódulo (repo independiente)
│   └── SCHEMA-COMPARTIDO/       ← mismo repo, cada servicio a su commit
├── docker-compose.yml           ← producción
├── docker-compose.dev.yml       ← desarrollo
├── .env / .env.example          ← puertos y variables globales
└── .gitignore
```

| Pieza | Responsabilidad | Guía |
|---|---|---|
| Repo raíz | Fija qué commit de cada servicio está en producción. Orquesta con Docker. | este documento |
| Submódulos | Código de cada servicio, con ciclo de vida propio. | [submódulos](../git/submodulos.md) |
| Dockerfiles | Cómo se construye cada servicio (dev y prod). | [dockerfiles](../docker/dockerfiles.md) |
| Compose + `.env` | Cómo se levantan y conectan los servicios. | [docker-compose](../docker/docker-compose.md) |
| Plantilla lista | Maqueta del repo raíz para copiar y adaptar. | [plantilla/](../docker/plantilla/README.md) |

---

## 2. ¿Por qué este patrón?

- **Despliegues reproducibles.** El raíz fija un commit exacto por servicio. Clonar el raíz en un commit dado trae siempre la misma combinación de versiones.
- **Ciclos de vida independientes.** Cada equipo trabaja su servicio con sus ramas y releases sin pisar a los demás.
- **Orquestación de un comando.** `docker compose up` levanta todo el stack, en dev o prod, con la misma topología.
- **Recursos compartidos versionados.** Un schema o librería común vive en un solo repo y cada servicio lo fija a la versión que le sirve.

### Cuándo NO usarlo

- Proyecto de un solo servicio → un repo normal basta.
- Equipo pequeño que despliega todo junto siempre → un monorepo simple es más fácil que coordinar submódulos.
- Los submódulos añaden fricción (detached HEAD, registrar referencias). Vale la pena cuando hay **varios servicios con releases independientes**.

---

## 3. Flujo de trabajo diario

1. **Clonar todo:** `git clone --recurse-submodules <url-raíz>`
2. **Levantar dev:** `docker compose -f docker-compose.dev.yml up`
3. **Trabajar en un servicio:** entrar al submódulo, `git checkout main`, cambios, commit, push (ver [submódulos §7](../git/submodulos.md)).
4. **Registrar la nueva versión en el raíz:** `git add <servicio> && git commit && git push`.
5. **Desplegar:** en el servidor, `git pull --recurse-submodules` y `docker compose up -d --build`.

---

## 4. Checklist — agregar un nuevo servicio

```
[ ] Crear repo del servicio en la organización de GitHub
[ ] git submodule add <url> <nombre>   (desde el repo raíz)
[ ] Crear Dockerfile (producción, multi-stage)
[ ] Crear Dockerfile.dev (desarrollo, single-stage)
[ ] Crear .dockerignore en el submódulo
[ ] Si necesita recurso compartido: git submodule add <schema-repo> dentro del submódulo
[ ] Agregar el servicio a docker-compose.dev.yml (bind mount + volumen node_modules)
[ ] Agregar el servicio a docker-compose.yml (límites de recursos)
[ ] Agregar el puerto a .env.example del raíz y al .env local
[ ] Crear .env.example del servicio (plantilla sin secretos)
[ ] git add + commit en el raíz con la nueva referencia del submódulo
```

---

## 5. Checklist — desplegar en un servidor

```
[ ] Acceso por SSH configurado (ver docs/ssh)
[ ] git clone --recurse-submodules <url-raíz>   (o git pull --recurse-submodules)
[ ] Crear .env del raíz a partir de .env.example
[ ] Crear .env de cada submódulo a partir de su .env.example
[ ] docker compose build
[ ] docker compose up -d
[ ] docker compose logs -f   (verificar arranque)
[ ] docker stats             (verificar recursos)
```

---

## 6. Errores comunes del patrón

| Síntoma | Causa | Solución |
|---|---|---|
| Submódulos vacíos tras clonar | Faltó `--recurse-submodules` | `git submodule update --init --recursive` |
| Cambios de un servicio no llegan al deploy | No se registró la referencia en el raíz | `git add <servicio> && git commit && git push` en el raíz |
| Servicio en "detached HEAD" | Normal tras `update --remote` | `git checkout main` dentro del submódulo |
| Un contenedor no ve a otro por `localhost` | `localhost` apunta al propio contenedor | Usar el nombre del servicio en la red (`http://backend:3000`) |
| Hot-reload muerto en Linux/WSL | El watcher no recibe eventos del bind mount | `CHOKIDAR_USEPOLLING` / `WATCHPACK_POLLING` |

Más detalle de cada caso en las guías de [submódulos](../git/submodulos.md) y [docker-compose](../docker/docker-compose.md).
