# 📘 Playbook de infraestructura y flujo de trabajo Git

Base de conocimiento práctica con guías paso a paso sobre **conexión a servidores, organización de repositorios e infraestructura con Docker**. Cada guía documenta, con ejemplos y explicaciones claras, cómo se ejecutan estos procesos en un entorno profesional real.

> Repositorio de **documentación**, no de código. Su valor es servir de referencia y dejar constancia del aprendizaje.

---

## 🗂️ Contenido

### 🔐 SSH
- [Gestión de claves SSH](docs/ssh/gestion-claves-ssh.md) — crear, usar, auditar y revocar claves; conexión a servidores y a GitHub; `~/.ssh/config`; diagnóstico de errores.

### 🌱 Git
- [Submódulos de Git](docs/git/submodulos.md) — organizar un proyecto en un repo raíz que orquesta servicios independientes; agregar, actualizar y eliminar submódulos (incluidos los anidados).
- [Clonar y migrar repositorios](docs/git/clonar-y-migrar-repositorios.md) — crear un repo nuevo a partir de uno existente, con o sin historial; migración exacta por espejo.

### 🐳 Docker
- [Dockerfiles por tipo de servicio](docs/docker/dockerfiles.md) — plantillas multi-stage (prod) y single-stage con hot-reload (dev) para backend y frontend.
- [Docker Compose](docs/docker/docker-compose.md) — orquestar todos los servicios, separar dev/prod, redes, volúmenes y variables de entorno.
- [Plantilla del repo raíz](docs/docker/plantilla/README.md) — maqueta lista para copiar (compose dev/prod + `.env.example`) con marcadores para adaptar.

### 🏗️ Arquitectura
- [Orquestación de servicios con submódulos + Docker](docs/arquitectura/orquestacion-submodulos-docker.md) — visión global del patrón, cuándo usarlo, flujo de trabajo y checklists para agregar servicios y desplegar.

### 🤖 Prompts
- [Prompts reutilizables](prompts/README.md) — plantillas para generar commits y solicitar implementaciones con asistentes de IA.

---

## 🧭 ¿Por dónde empezar?

- **Quiero conectarme a un servidor** → [SSH](docs/ssh/gestion-claves-ssh.md)
- **Quiero entender la estructura del proyecto** → [Arquitectura](docs/arquitectura/orquestacion-submodulos-docker.md)
- **Voy a trabajar con submódulos** → [Submódulos](docs/git/submodulos.md)
- **Voy a levantar el entorno** → [Docker Compose](docs/docker/docker-compose.md)
- **Voy a migrar o duplicar un repo** → [Clonar y migrar](docs/git/clonar-y-migrar-repositorios.md)

---

## 📁 Estructura del repositorio

```
.
├── README.md
├── docs/
│   ├── ssh/
│   │   └── gestion-claves-ssh.md
│   ├── git/
│   │   ├── submodulos.md
│   │   └── clonar-y-migrar-repositorios.md
│   ├── docker/
│   │   ├── dockerfiles.md
│   │   ├── docker-compose.md
│   │   └── plantilla/                       # maqueta lista para copiar
│   │       ├── README.md
│   │       ├── docker-compose.yml
│   │       ├── docker-compose.dev.yml
│   │       └── .env.example
│   └── arquitectura/
│       └── orquestacion-submodulos-docker.md
└── prompts/
    ├── README.md
    ├── commit-completo.md
    ├── commit-staging.md
    └── solicitud-cambios.md
```
