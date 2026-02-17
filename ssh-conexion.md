# Guía práctica para gestión de claves SSH (local & servidor)

---

## 🚩 ¿Dónde se hace cada paso?

| Acción                     | Lugar       |
| -------------------------- | ----------- |
| Generar claves             | 💻 Local    |
| Copiar clave pública       | 💻 Local    |
| Conectarse por SSH         | 💻 Local    |
| Autorizar / revocar claves | 🖥️ Servidor |
| Borrar clave privada       | 💻 Local    |

---

## 🔒 Algoritmo

```bash
ed25519
```

---

## 🏷️ Nomenclatura

```text
id_ed25519_<proyecto_o_servidor>
```

Ejemplo:

```text
id_ed25519_servidorweb
```

---

# 💻 LOCAL

## 1️⃣ Crear clave

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_servidorweb -C "usuario@servidor"
```

- Elige una passphrase segura cuando se te pida (opcional pero recomendable).

---

## 2️⃣ Copiar la clave al servidor

Puerto por defecto:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519_servidorweb.pub usuario@servidor
```

Con puerto distinto:

```bash
ssh-copy-id -p 2222 -i ~/.ssh/id_ed25519_servidorweb.pub usuario@servidor
```

---

## 3️⃣ Conectarse usando esa clave

```bash
ssh -i ~/.ssh/id_ed25519_servidorweb usuario@servidor
```

Con puerto:

```bash
ssh -p 2222 -i ~/.ssh/id_ed25519_servidorweb usuario@servidor
```

---

## 4️⃣ Eliminar una clave creada por error (local)

```bash
rm ~/.ssh/id_ed25519_servidorweb
rm ~/.ssh/id_ed25519_servidorweb.pub
```

> ⚠️ Esto no revoca el acceso en el servidor.

---

# 🖥️ SERVIDOR

## 5️⃣ Archivo de claves autorizadas

```text
~/.ssh/authorized_keys
```

---

## 6️⃣ Ver claves autorizadas

```bash
cat ~/.ssh/authorized_keys
```

---

## 7️⃣ Revocar acceso

```bash
nano ~/.ssh/authorized_keys
```

Eliminar únicamente la línea correspondiente.

---

## 8️⃣ Identificar qué línea borrar

Desde local:

```bash
cat ~/.ssh/id_ed25519_servidorweb.pub
```

Buscar esa línea exacta en `authorized_keys`.

---

# 🚨 Flujo correcto para revocar una clave

1️⃣ En el servidor → borrar la línea en `authorized_keys`
2️⃣ En local → borrar los archivos con `rm`

---

# 🧠 Validación importante (caso real)

### Error típico

```text
Too many authentication failures
```

### Causa

Tu equipo tiene muchas llaves cargadas en el agente SSH y el servidor corta la conexión tras varios intentos.

### Solución (usar solo la clave correcta)

Para copiar la clave:

```bash
ssh-copy-id -p 2222 -i ~/.ssh/id_ed25519_servidorweb.pub -o IdentitiesOnly=yes usuario@servidor
```

Para conectarte:

```bash
ssh -p 2222 -i ~/.ssh/id_ed25519_servidorweb -o IdentitiesOnly=yes usuario@servidor
```

👉 Esta opción obliga a SSH a usar únicamente esa clave.

---

# ❗ Errores que debes evitar

- No subir nunca la clave privada a repositorios.
- No copiar la clave privada al servidor.
- No reutilizar la misma clave para varios servidores.

---

# 🥇 Regla de oro

Una clave por servidor o entorno:

```text
id_ed25519_backend_prod
id_ed25519_frontend_dev
id_ed25519_clienteX
```
