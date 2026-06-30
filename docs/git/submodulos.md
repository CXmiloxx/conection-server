# Submódulos de Git: organización de proyectos por servicios

Cómo estructurar un proyecto en un **repositorio raíz** que orquesta varios **servicios independientes** mediante git submodules. Cada servicio (backend, frontend, infra, schema compartido) vive en su propio repositorio con su ciclo de vida; el repo raíz solo fija **qué commit** de cada uno está en producción.

> Para la parte de Docker (cómo se levantan esos servicios) ver [docker-compose](../docker/docker-compose.md) y [dockerfiles](../docker/dockerfiles.md). Para la visión global del patrón ver [arquitectura](../arquitectura/orquestacion-submodulos-docker.md).

---

## 1. ¿Qué es un submódulo?

Un submódulo es un repositorio Git **anidado dentro de otro**. El repo raíz no guarda el código del submódulo: guarda una **referencia a un commit exacto** de ese otro repo, más un archivo `.gitmodules` con la URL y la ruta.

Consecuencia clave: un submódulo **apunta a un commit fijo, no a una rama**. Avanza solo cuando lo actualizas explícitamente y registras la nueva referencia en el repo raíz. Esto da despliegues **reproducibles**: clonar el raíz en un commit dado siempre trae los mismos commits de cada servicio.

---

## 2. Estructura recomendada

El repositorio raíz se encarga **solo de la orquestación**: nada de código de aplicación.

```
ORG-RAIZ/                        ← repo orquestador
│   Sin código de aplicación.
│   Solo: docker-compose + .env + submódulos.
│
├── BACKEND/                     ← submódulo (repo independiente)
├── FRONTEND/                    ← submódulo (repo independiente)
├── SCHEMA-COMPARTIDO/           ← submódulo (recursos compartidos)
├── docker-compose.yml           ← producción
├── docker-compose.dev.yml       ← desarrollo
├── .gitignore
└── .env.example                 ← puertos y variables globales
```

| Repo | Responsabilidad |
|---|---|
| **Raíz** | Fija qué commit de cada submódulo está en producción. Orquesta con Docker. |
| **Submódulo** | Código de aplicación con su propio ciclo de vida, ramas y releases. |

> 👉 **NO** guardes código fuente de los servicios en el repo raíz.
> 👉 El repo raíz facilita la orquestación y el despliegue conjunto.

### Submódulo anidado (recurso compartido)

Cuando varios servicios comparten un recurso (schema de DB, librería de tipos, utilidades), ese recurso vive en su propio repo y cada servicio lo incluye como **submódulo anidado**. Cada servicio puede apuntarlo a un commit distinto.

```
ORG-RAIZ/
├── BACKEND/
│   └── SCHEMA-COMPARTIDO/    ← submódulo anidado
├── FRONTEND/
│   └── SCHEMA-COMPARTIDO/    ← mismo repo, cada servicio lo fija a su commit
├── docker-compose.yml
└── docker-compose.dev.yml
```

---

## 3. Crear el repositorio raíz

```bash
mkdir ORG-RAIZ
cd ORG-RAIZ
git init
# Archivos de orquestación
git add docker-compose.yml docker-compose.dev.yml .env.example .gitignore
git commit -m "chore: init repo de orquestación"
git remote add origin https://github.com/ORG/ORG-RAIZ.git
git push -u origin main
```

---

## 4. Agregar submódulos

### Sintaxis general

```bash
git submodule add <url> <directorio>
```

Git crea el directorio, clona el repo dentro y registra el vínculo en `.gitmodules`:

```ini
[submodule "BACKEND"]
    path = BACKEND
    url  = https://github.com/ORG/PROYECTO_BACK.git
```

### Caso 1: el repo existe pero la carpeta local **aún no**

```bash
git submodule add https://github.com/ORG/PROYECTO_BACK.git BACKEND
git submodule add https://github.com/ORG/PROYECTO_FRONT.git FRONTEND
```

Git crea las carpetas y descarga el código.

### Caso 2: la carpeta local **ya existe**

Usa `-f` para sobrescribir el registro de una carpeta existente:

```bash
git submodule add -f https://github.com/ORG/PROYECTO_BACK.git BACKEND
```

### Guardar y subir la relación

Cada vez que agregas o cambias submódulos hay que registrar el cambio en el repo raíz:

```bash
git add .gitmodules BACKEND FRONTEND
git commit -m "chore(modules): agregar submódulos backend y frontend"
git push
```

### Submódulo anidado (dentro de un servicio)

```bash
cd BACKEND
git submodule add https://github.com/ORG/SCHEMA-COMPARTIDO.git SCHEMA-COMPARTIDO
git add .gitmodules SCHEMA-COMPARTIDO
git commit -m "chore(modules): agregar submódulo SCHEMA-COMPARTIDO"
git push

# Volver al raíz y registrar la nueva referencia del backend
cd ..
git add BACKEND
git commit -m "chore(modules): actualiza ref de BACKEND con SCHEMA-COMPARTIDO"
git push
```

---

## 5. Clonar un proyecto con submódulos

```bash
# Recomendado: clonar e inicializar todo de una vez
git clone --recurse-submodules https://github.com/ORG/ORG-RAIZ.git
```

Si ya clonaste sin el flag y las carpetas de submódulos están vacías:

```bash
git submodule update --init --recursive
# --init       inicializa submódulos nuevos
# --recursive  también inicializa los anidados
```

---

## 6. Actualizar submódulos

Como apuntan a un commit fijo, traer los últimos cambios es **explícito**.

```bash
# Actualizar uno al último commit de su rama remota (main por defecto)
git submodule update --remote BACKEND

# Actualizar todos, mezclando y de forma recursiva
git submodule update --remote --merge --recursive
```

Después hay que **registrar la nueva referencia** en el repo raíz, o el cambio no queda guardado:

```bash
git add BACKEND FRONTEND
git commit -m "chore(modules): actualiza submódulos al último commit"
git push
```

### Pull recursivo

```bash
git pull --recurse-submodules

# Hacerlo el comportamiento por defecto
git config --global submodule.recurse true
```

---

## 7. Trabajar dentro de un submódulo

Tras actualizar, un submódulo queda en **detached HEAD** (apunta a un commit suelto, no a una rama). Antes de trabajar, haz checkout a la rama:

```bash
cd BACKEND
git checkout main
git pull origin main

# Cambios normales
git add .
git commit -m "feat: nueva funcionalidad"
git push origin main

# Volver al raíz y registrar el nuevo commit del submódulo
cd ..
git add BACKEND
git commit -m "chore(modules): actualiza BACKEND"
git push
```

> 👉 **Obligatorio:** el último paso asegura que el repo raíz apunte al commit recién subido. Si lo omites, otra persona que clone el raíz no verá tus cambios.

---

## 8. Ver el estado de los submódulos

```bash
git submodule status
# +abc1234 BACKEND    ← + hay commits nuevos disponibles (raíz desactualizado)
#  def5678 FRONTEND   ← sin prefijo = en el commit esperado
# -ghi9012 SCHEMA     ← - no está inicializado (falta update --init)
```

Ejecutar un comando en cada submódulo:

```bash
git submodule foreach git status     # estado git de cada uno
git submodule foreach git branch     # rama actual de cada uno
```

---

## 9. Eliminar un submódulo

```bash
git submodule deinit -f BACKEND        # quita la copia de trabajo
git rm -f BACKEND                      # quita el registro y .gitmodules
rm -rf .git/modules/BACKEND            # limpia el repo interno cacheado
git commit -m "chore(modules): eliminar submódulo BACKEND"
git push
```

Los tres primeros pasos son necesarios: omitir `rm -rf .git/modules/...` deja basura que rompe un futuro `submodule add` con el mismo nombre.

---

## 10. Troubleshooting

### Submódulo vacío después de clonar

Clonaste sin `--recurse-submodules`:

```bash
git submodule update --init --recursive
```

### Submódulo en "detached HEAD"

Normal tras `update`. Para trabajar, vuelve a una rama:

```bash
cd BACKEND
git checkout main
git pull origin main
```

### Cambios en el submódulo que no aparecen al clonar el raíz

Olvidaste registrar la referencia en el raíz:

```bash
git add BACKEND && git commit -m "chore(modules): actualiza BACKEND" && git push
```

### El raíz marca el submódulo como modificado y no quieres ese cambio

```bash
git submodule update --init BACKEND     # vuelve al commit que fija el raíz
```

---

## 11. Referencia rápida

```bash
# Clonar con submódulos
git clone --recurse-submodules <url>

# Inicializar en repo ya clonado
git submodule update --init --recursive

# Agregar (-f si la carpeta ya existe)
git submodule add <url> <directorio>

# Actualizar uno / todos
git submodule update --remote <directorio>
git submodule update --remote --merge --recursive

# Registrar la nueva referencia en el raíz
git add <directorio> && git commit -m "chore(modules): actualiza <dir>" && git push

# Estado
git submodule status
git submodule foreach <comando>

# Pull recursivo por defecto
git config --global submodule.recurse true

# Eliminar
git submodule deinit -f <dir>
git rm -f <dir>
rm -rf .git/modules/<dir>
```
