# Crear un repositorio nuevo a partir de uno existente

Cómo crear un repositorio remoto nuevo (GitHub, GitLab, Bitbucket…) partiendo de un proyecto existente, **sin modificar ni poner en riesgo el original**.

Útil cuando quieres:

- Reutilizar código como base de un proyecto nuevo.
- Migrar un repo a otra organización o plataforma.
- Reorganizar o separar partes de un proyecto en repos independientes.
- Conservar o descartar el historial, según el caso.

---

## Decidir: ¿conservar el historial o empezar limpio?

| | Escenario 1 — Conservar historial | Escenario 2 — Sin historial |
|---|---|---|
| **Qué copia** | Todos los commits, ramas y etiquetas | Solo el estado actual de los archivos |
| **Cuándo usarlo** | Migraciones, bifurcaciones, duplicados con trazabilidad | "Fresh start", plantillas, cuando el historial es ruido |
| **Reversible** | El historial queda disponible | El historial anterior se pierde |

> 🧩 **Recomendación:** para proyectos reales (frontend moderno, backend Node/Nest, Prisma, monorepos) casi siempre conviene **conservar el historial** (Escenario 1): mantiene trazabilidad de bugs, permite `git revert` y ayuda al onboarding.

---

## ✅ Escenario 1 — Conservar todo el historial

### 1. Clonar el repositorio original

```bash
git clone <URL_REPO_ORIGINAL>
cd <nombre-del-repo>
```

- La URL puede ser HTTPS o SSH.
- Para incluir además todo el historial de **todas** las ramas remotas, usa un clon espejo (ver más abajo).

### 2. Eliminar la referencia al remoto original

```bash
git remote remove origin
```

Esto solo afecta tu copia local. El repositorio original sigue intacto en su servidor y para otros colaboradores.

### 3. Crear un repositorio vacío en la plataforma remota

En GitHub/GitLab/Bitbucket crea un repo nuevo **sin inicializar**: NO marques “Add README”, “Add .gitignore” ni “Add License”. Si lo inicializas, el push fallará (ver Problemas comunes #2).

Copia la URL del nuevo repo, p. ej. `https://github.com/TU-ORG/REPO-NUEVO.git`.

### 4. Asociar el local al nuevo remoto

```bash
git remote add origin <URL_REPO_NUEVO>
```

### 5. Subir el historial

```bash
git push -u origin main
```

- Si tu rama principal es `master` u otra, ajústalo.
- `-u` deja `origin/main` como rama de seguimiento para futuros `push`/`pull`.

**Opcional — subir todas las ramas y etiquetas:**

```bash
git push --all origin
git push --tags
```

### 6. Verificar

```bash
git log --oneline --decorate --all     # historial local
```

Revisa en la plataforma que ramas, tags e historial se transfirieron.

> ✔ El original **no se modifica**. El nuevo repo tiene el código y el historial completo.

### Variante: migración exacta (clon espejo)

Para mover **absolutamente todo** (todas las ramas, tags y refs) a otra plataforma, lo más limpio es un clon espejo:

```bash
git clone --mirror <URL_REPO_ORIGINAL>
cd <repo>.git
git remote set-url origin <URL_REPO_NUEVO>
git push --mirror
```

`--mirror` replica el repositorio tal cual. Ideal para cambiar de proveedor sin perder nada.

---

## ✅ Escenario 2 — Empezar SIN historial

Deja atrás todos los commits antiguos y copia solo el estado actual de los archivos.

### 1. Ubicarse en la raíz del proyecto

```bash
cd <carpeta-del-proyecto>
```

### 2. Eliminar la carpeta de control de versiones

```bash
rm -rf .git
```

> ⚠️ Esto **borra todo el historial** local de forma irreversible. Asegúrate de no necesitarlo (o haz una copia de la carpeta antes).

### 3. Inicializar un repositorio nuevo

```bash
git init
```

### 4. Asociar el nuevo remoto

Crea el repo en la plataforma **sin README, sin .gitignore, sin licencia** y asócialo:

```bash
git remote add origin <URL_REPO_NUEVO>
```

### 5. Primer commit

```bash
git status            # revisa qué se va a incluir
git add .
git commit -m "Initial commit"
```

> 💡 Crea un `.gitignore` **antes** de este paso para no versionar `node_modules`, `.env`, builds, etc.

### 6. Subir al remoto

```bash
git push -u origin main
```

> ✔ El nuevo repo solo tiene los archivos actuales, desde un commit inicial. El historial anterior desaparece de este repo.

---

## ⚠️ Problemas comunes

### 1. Eliminar el remoto NO borra el repositorio original

```bash
git remote remove origin
```

Solo elimina la **referencia local**. No toca el repo remoto ni el de otros colaboradores.

### 2. Error `rejected – non-fast-forward` al hacer push

```text
! [rejected]        main -> main (non-fast-forward)
error: failed to push some refs to 'https://github.com/TU-ORG/REPO-NUEVO.git'
```

Ocurre porque el repo nuevo **no está vacío** (lo inicializaron con README/licencia). Opciones:

```bash
# A) Integrar ambos historiales no relacionados
git pull origin main --allow-unrelated-histories
git push

# B) (preferible) Borrar el repo remoto y recrearlo completamente vacío, luego push
```

### 3. `git status` da error

Probablemente no ejecutaste `git init` o borraste `.git` por accidente. Reinicializa con `git init`.

### 4. Archivos ocultos y configuraciones previas

Revisa carpetas ocultas (`.github/`, `.vscode/`, `.idea/`, configs de CI). Decide si las incluyes o las añades a `.gitignore` antes del primer commit.

---

## 📌 Resumen rápido

**Conservar TODO el historial (recomendado):**

```bash
git clone <repo-viejo>
cd <carpeta-clonada>
git remote remove origin
git remote add origin <repo-nuevo>
git push -u origin main
# Opcional: todas las ramas y tags
git push --all origin
git push --tags
```

**Migración exacta (espejo):**

```bash
git clone --mirror <repo-viejo>
cd <repo>.git
git remote set-url origin <repo-nuevo>
git push --mirror
```

**Empezar SIN historial:**

```bash
rm -rf .git
git init
git remote add origin <repo-nuevo>
git add .
git commit -m "Initial commit"
git push -u origin main
```

---

## 📝 Buenas prácticas

- Verifica **antes** de eliminar `.git` o referencias remotas.
- Haz respaldo si el historial importa, antes de comandos destructivos.
- En equipos, **comunica y coordina** la migración para evitar pérdidas de trabajo.
- Configura ramas protegidas y permisos en el nuevo repo si será colaborativo.
