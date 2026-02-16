# Guía para Gestión de Claves SSH (local & servidor)

---

## 🚩 Resumen: ¿Dónde se hace cada paso?

| Acción                               | ¿Dónde?     |
| ------------------------------------ | ----------- |
| **Generar pares de claves**          | 💻 Local    |
| **Copiar clave pública al servidor** | 💻 Local    |
| **Conectarse al servidor**           | 💻 Local    |
| **Autorizar o revocar accesos**      | 🖥️ Servidor |
| **Borrar claves privadas**           | 💻 Local    |

---

## 🔒 Algoritmo Recomendado

**Usa SIEMPRE:**

```bash
ed25519
```

_Más seguro y rápido que RSA._

---

## 🏷️ Nomenclatura de las claves

Nombre sugerido para identificar cada par:

```text
id_ed25519_<proyecto_o_servidor>
```

Ejemplo:

```text
id_ed25519_servidorweb
```

¡Así sabrás siempre para qué sirve cada clave!

---

# 💻 PASOS EN LOCAL

## 1️⃣ Generar una nueva clave SSH

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_servidorweb -C "tuusuario@servidor"
```

- Elige una passphrase segura cuando se te pida (opcional pero recomendable).

---

## 2️⃣ Copiar tu clave pública al servidor

```bash
ssh-copy-id -i ~/.ssh/id_ed25519_servidorweb.pub usuario@servidor
```

¿Servidor en puerto distinto? Usa:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519_servidorweb.pub -p 2222 usuario@servidor
```

---

## 3️⃣ Conectarse usando tu clave

```bash
ssh -i ~/.ssh/id_ed25519_servidorweb usuario@servidor
```

- ¿Otro puerto? Añade `-p <puerto>`.

---

## 4️⃣ Eliminar una clave creada por error (local)

```bash
rm ~/.ssh/id_ed25519_servidorweb
rm ~/.ssh/id_ed25519_servidorweb.pub
```

> ⚠️ **Esto sólo elimina la copia local. Si ya la copiaste al servidor, también debes borrarla allí.**

---

# 🖥️ PASOS EN SERVIDOR

## 5️⃣ Dónde están las claves autorizadas

Tu archivo de claves públicas autorizadas es:

```text
~/.ssh/authorized_keys
```

---

## 6️⃣ Ver las claves autorizadas

```bash
cat ~/.ssh/authorized_keys
```

---

## 7️⃣ Revocar acceso (quitar una clave)

Edita el archivo:

```bash
nano ~/.ssh/authorized_keys
```

- Borra la línea correspondiente a la clave a revocar (¡cuidado!).

---

## 8️⃣ ¿Cuál línea borrar?

Desde local puedes ver el contenido de tu clave pública:

```bash
cat ~/.ssh/id_ed25519_servidorweb.pub
```

Busca exactamente esa línea dentro de `authorized_keys` en el servidor. Solo borra esa línea.

---

# 🚨 ¿Borraste la clave por accidente o quieres quitar acceso?

**Procedimiento correcto:**

1. **En el servidor:** Borra la línea de la clave en `authorized_keys`.
2. **En local:** Borra ambos archivos de la clave con `rm`.

---

# ❗ Errores comunes (¡Evítalos siempre!)

- Nunca subas la **clave privada** a ningún repositorio ni compartas por ningún medio.
- Jamás copies la clave privada al servidor.
- Evita reutilizar la misma clave para diferentes servidores o servicios.

---

# 🥇 Regla de oro

**Crea y usa una clave diferente para cada servidor o entorno:**

```text
id_ed25519_backend_prod
id_ed25519_frontend_dev
id_ed25519_clienteX
```

¡Así tendrás seguridad y control total!

---
