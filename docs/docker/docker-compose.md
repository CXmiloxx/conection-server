# Docker Compose: orquestación y variables de entorno

Cómo el repo raíz levanta todos los servicios ([submódulos](../git/submodulos.md)) con un solo comando, separando **desarrollo** y **producción**, y cómo gestionar las variables de entorno. Los Dockerfiles que se construyen aquí están en [dockerfiles](./dockerfiles.md).

El repo raíz tiene dos archivos:

| Archivo | Uso |
|---|---|
| `docker-compose.yml` | Producción |
| `docker-compose.dev.yml` | Desarrollo (bind mounts + hot-reload) |

> 📦 ¿Buscas archivos listos para copiar? En [plantilla/](./plantilla/README.md) hay una maqueta completa (compose de dev y prod + `.env.example`) con marcadores (`org`, nombres de servicio) para adaptar a tu proyecto.

---

## 1. Servicio en desarrollo

```yaml
services:
  mi_servicio:
    container_name: mi_servicio_dev
    image: mi_servicio_dev
    build:
      context: ./BACKEND          # directorio del submódulo
      dockerfile: Dockerfile.dev
    restart: unless-stopped
    ports:
      - "${MI_SERVICIO_PORT:-3000}:3000"
    environment:
      NODE_ENV: development
      LOG_LEVEL: debug
      PORT: 3000
    env_file:
      - ./BACKEND/.env
    working_dir: /app
    volumes:
      - ./BACKEND:/app
      # node_modules en volumen nombrado para que el bind mount no los pise
      - mi_servicio_node_modules:/app/node_modules
      # si el servicio genera artefactos de un submódulo anidado
      - mi_servicio_generated:/app/SCHEMA-COMPARTIDO/generated
    networks:
      - app_network
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

volumes:
  mi_servicio_node_modules:
  mi_servicio_generated:

networks:
  app_network:
    driver: bridge
    name: mi_org_dev_network
```

### Por qué `node_modules` va en volumen nombrado

El bind mount `./BACKEND:/app` monta el directorio del **host** sobre `/app` del contenedor, incluyendo los `node_modules` del host. Esos módulos pueden estar compilados para otra arquitectura o sistema operativo.

El volumen nombrado en `/app/node_modules` **tiene prioridad** sobre el bind mount en ese path específico: el contenedor usa sus propios módulos compilados para Linux/Alpine y el host no los sobreescribe.

Si el servicio usa un submódulo anidado que genera artefactos (ej. cliente Prisma), ese directorio de generación va también en volumen nombrado por la misma razón.

---

## 2. Servicio en producción

```yaml
services:
  mi_servicio:
    container_name: mi_servicio_prod
    build:
      context: ./BACKEND
      dockerfile: Dockerfile
    image: mi_servicio_prod
    restart: unless-stopped
    ports:
      - "${MI_SERVICIO_PORT:-3000}:3000"
    env_file:
      - ./BACKEND/.env
    environment:
      NODE_ENV: production
      LOG_LEVEL: info
      PORT: 3000
    networks:
      - app_network
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
    logging:
      driver: "json-file"
      options:
        max-size: "50m"
        max-file: "5"

networks:
  app_network:
    driver: bridge
    name: mi_org_prod_network
```

---

## 3. Diferencias dev vs prod

| | Desarrollo | Producción |
|---|---|---|
| Código | Bind mount desde el host | Copiado en el build |
| `node_modules` | Volumen nombrado separado | Dentro de la imagen |
| Imágenes | Single-stage | Multi-stage (imagen mínima) |
| Límites de recursos | No | Sí (cpu + memoria) |
| Logs | 10MB × 3 archivos | 50MB × 5 archivos |
| `LOG_LEVEL` | `debug` | `info` |

---

## 4. Comunicación entre servicios

Los contenedores en la misma red (`app_network`) se resuelven **por nombre de servicio**, no por `localhost`. Desde `frontend`, el backend está en:

```text
http://backend:3000        ← nombre del servicio + puerto interno
```

`localhost` dentro de un contenedor se refiere a sí mismo, no al host ni a otros contenedores. El puerto publicado (`ports:`) solo importa para acceder **desde el host**; entre contenedores se usa el puerto interno.

---

## 5. Variables de entorno

### Estructura de archivos

```
ORG-RAIZ/
├── .env                 ← puertos y vars del orquestador (NO versionado)
├── .env.example         ← plantilla (versionada, sin secretos)
│
├── BACKEND/
│   ├── .env             ← secretos del servicio (NO versionado)
│   └── .env.example     ← plantilla del servicio (versionada)
│
└── FRONTEND/
    ├── .env
    └── .env.example
```

> 🔒 Los `.env` con secretos **nunca** se versionan. Solo se versionan los `.env.example` como plantilla. Asegúrate de que `.gitignore` incluya `.env`.

### `.env.example` del repo raíz

Solo puertos y configuración de orquestación:

```env
SERVICIO_A_PORT=3000
SERVICIO_B_PORT=3001

NODE_ENV=development
```

Referenciados en compose con valor por defecto (`:-`):

```yaml
ports:
  - "${SERVICIO_A_PORT:-3000}:3000"
```

`${VAR:-default}` usa `VAR` si está definida; si no, `default`. Así el proyecto arranca aunque falte el `.env`.

### `.env` de cada submódulo

Secretos y configuración específica del servicio:

```env
DATABASE_URL="mysql://user:pass@host:3306/db"
JWT_SECRET="..."
PORT=3000
```

---

## 6. Comandos

### Desarrollo

```bash
docker compose -f docker-compose.dev.yml up           # levantar (foreground)
docker compose -f docker-compose.dev.yml up -d        # en segundo plano
docker compose -f docker-compose.dev.yml up <servicio># solo uno
docker compose -f docker-compose.dev.yml build <servicio>
docker compose -f docker-compose.dev.yml logs -f <servicio>
docker compose -f docker-compose.dev.yml exec <servicio> sh   # shell dentro del contenedor
docker compose -f docker-compose.dev.yml down         # detener y eliminar contenedores
docker compose -f docker-compose.dev.yml down -v      # + eliminar volúmenes
```

### Producción

```bash
docker compose up -d
docker compose build <servicio>
docker compose up -d <servicio>
docker compose logs -f <servicio>
docker compose down
docker stats                       # uso de cpu/memoria en vivo
```

### Limpieza

```bash
docker image prune                 # imágenes sin uso
docker system prune -a --volumes   # limpieza total (cuidado: borra volúmenes)
```

---

## 7. Troubleshooting

### Hot-reload no detecta cambios (Linux / WSL / Docker Desktop)

El watcher no recibe eventos del filesystem montado. Forzar polling:

```yaml
environment:
  WATCHPACK_POLLING: "true"      # Next.js
  CHOKIDAR_USEPOLLING: "true"    # NestJS / otros watchers
```

### Puerto en uso

```bash
lsof -i :<puerto>                  # ver qué proceso lo ocupa
# Cambiar el puerto en .env (NO en docker-compose)
SERVICIO_A_PORT=3010
```

### `node_modules` desincronizados tras cambiar `package.json`

```bash
docker compose -f docker-compose.dev.yml build <servicio>
docker compose -f docker-compose.dev.yml up <servicio>
```

### Reset completo de un servicio

```bash
docker compose -f docker-compose.dev.yml down -v
docker compose -f docker-compose.dev.yml build --no-cache <servicio>
docker compose -f docker-compose.dev.yml up <servicio>
```

### Un servicio arranca antes que su dependencia (ej. la DB)

`depends_on` espera a que el contenedor **arranque**, no a que esté **listo**. Para esperar a que esté sano, usa `condition: service_healthy` junto a un `healthcheck`:

```yaml
  backend:
    depends_on:
      db:
        condition: service_healthy
```
