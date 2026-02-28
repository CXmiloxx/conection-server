Aquí tienes una **guía clara y completa** sobre cómo organizar y trabajar con repositorios usando submódulos de Git en GitHub, ideal para equipos de desarrollo con estructuras divididas entre infraestructura y código de app.

---

# 📦 Estructura recomendada del repositorio principal (“repo grande”)

El repositorio principal se encarga **solo de la infraestructura y la configuración global**. El código de las aplicaciones (backend y frontend) se mantiene en subrepositorios independientes, conectados como submódulos.

```
PROYECTO_INFRA/
├── docker-compose.yml   # Configuración de contenedores
├── .env                # Variables de entorno globales
├── .gitignore
├── BACKEND/            # Submódulo: código backend (otro repo)
└── FRONTEND/           # Submódulo: código frontend (otro repo)
```

👉 Importante: **NO guardes código fuente de los proyectos en el repo grande.**
👉 El repo principal facilita la orquestación y el despliegue conjunto.

---

# 🧱 Pasos para crear el repositorio principal en la organización

1. **Crea la carpeta y navega a ella:**
   ```bash
   mkdir PROYECTO_INFRA
   cd PROYECTO_INFRA
   ```
2. **Inicializa un nuevo repositorio Git y agrega los archivos principales:**
   ```bash
   git init
   git add docker-compose.yml .env .gitignore
   git commit -m "init infra repo"
   ```
3. **Conecta el repo local con tu organización en GitHub y realiza el primer push:**
   ```bash
   git remote add origin https://github.com/ORG/PROYECTO_INFRA.git
   git push -u origin master
   ```

---

# 🔗 Cómo agregar subrepositorios (submódulos)

## ✅ Caso 1: Los subrepos YA existen **y las carpetas locales también**

Si ya tienes una copia local de los subrepositorios en las carpetas `BACKEND` y `FRONTEND`, usa:

```bash
git submodule add -f https://github.com/ORG/PROYECTO_BACK.git BACKEND
git submodule add -f https://github.com/ORG/PROYECTO_FRONT.git FRONTEND
```

El flag `-f` es necesario para sobrescribir la carpeta si ya existe.

---

## ✅ Caso 2: Los subrepos YA existen **pero NO están clonados todavía**

Si las carpetas aún no existen localmente, ejecuta:

```bash
git submodule add https://github.com/ORG/PROYECTO_BACK.git BACKEND
git submodule add https://github.com/ORG/PROYECTO_FRONT.git FRONTEND
```

Git creará las carpetas y cargará el código desde los repos originales.

---

## ✅ Guarda y sube la relación de submódulos

Cada vez que agregas o actualizas submódulos debes actualizar el repo grande:

```bash
git commit -m "add subrepositories"
git push
```
Esto guardará la referencia de los submódulos en `.gitmodules` y en el árbol de commits.

---

# 🔁 Cómo actualizar el repo grande después de cambios en un subrepo

Cuando realices cambios en un subrepositorio, asegúrate de que el repo grande apunte al commit actualizado del subrepo. Hazlo así:

## 1️⃣ Realiza los cambios en el subrepo:

```bash
cd BACKEND
git add .
git commit -m "describe tus cambios"
git push
```

## 2️⃣ Actualiza el submódulo en el repo grande:

Sal un nivel atrás al repo grande, luego:

```bash
cd ..
git add BACKEND
git commit -m "update backend submodule"
git push
```

👉 **Obligatorio:** Este paso asegura que el repo grande apunte al último commit actualizado del subrepo.

---

## 🔄 Actualizar todos los subrepositorios a la vez

Para obtener los últimos cambios de todos los submódulos desde sus repositorios remotos y mezclarlos, ejecuta:

```bash
git submodule update --remote --merge --recursive
```

Luego haz commit en el repo grande para registrar estos nuevos commits de los subrepositorios:

```bash
git add BACKEND FRONTEND
git commit -m "update all submodules"
git push
```

---

# ✅ Resumen rápido paso a paso

**Para crear el repo grande:**
```
git init
git add docker-compose.yml .env .gitignore
git commit
git push
```

**Para agregar subrepositorios ya existentes:**
```
git submodule add <URL_SUBREPO> <CARPETA>
# Usa -f si la carpeta ya existe
```

**Para actualizar referencia de submódulo en el repo grande:**
```
git add SUBREPO
git commit
git push
```

**Para traer cambios de todos los subrepositorios:**
```
git submodule update --remote --merge --recursive
git add SUBREPO
git commit
git push
```