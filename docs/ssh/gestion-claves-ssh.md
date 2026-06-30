# Gestión de claves SSH (local y servidor)

Guía práctica para crear, usar, auditar y revocar claves SSH de forma segura. Pensada para conectarse a servidores remotos y a proveedores Git (GitHub, GitLab) sin contraseña y sin exponer secretos.

---

## 1. Conceptos básicos

Una clave SSH es un **par** de archivos:

| Archivo | Contenido | Dónde vive |
|---|---|---|
| `id_ed25519_<nombre>` | Clave **privada** | Solo en tu equipo local. **Nunca** se copia ni se sube. |
| `id_ed25519_<nombre>.pub` | Clave **pública** | Se copia al servidor (`authorized_keys`) o se pega en GitHub. |

El servidor guarda la clave **pública**. Tú conservas la **privada**. Al conectarte, SSH demuestra que posees la privada sin enviarla nunca por la red.

### ¿Dónde se hace cada paso?

| Acción | Lugar |
| --- | --- |
| Generar el par de claves | 💻 Local |
| Copiar la clave pública | 💻 Local |
| Conectarse por SSH | 💻 Local |
| Autorizar / revocar claves | 🖥️ Servidor |
| Borrar la clave privada | 💻 Local |

---

## 2. Algoritmo y nomenclatura recomendados

### Algoritmo

Usa **Ed25519**: más corto, más rápido y más seguro que RSA para el mismo nivel de protección.

```bash
ed25519
```

> Solo si el servidor es muy antiguo y no soporta Ed25519, usa RSA de 4096 bits:
> `ssh-keygen -t rsa -b 4096 ...`

### Nomenclatura

Un nombre por **servidor o entorno**. Facilita auditar y revocar sin afectar otros accesos.

```text
id_ed25519_<proyecto_o_servidor>
```

Ejemplos:

```text
id_ed25519_servidorweb
id_ed25519_backend_prod
id_ed25519_frontend_dev
id_ed25519_clienteX
id_ed25519_github
```

---

## 💻 LOCAL

### 3. Crear la clave

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_servidorweb -C "usuario@servidor"
```

| Flag | Significado |
|---|---|
| `-t ed25519` | Tipo de algoritmo. |
| `-f <ruta>` | Ruta y nombre del archivo de la clave privada (la `.pub` se crea sola). |
| `-C "<comentario>"` | Etiqueta que aparece al final de la pública. Útil para identificarla en `authorized_keys`. |

- Cuando pida **passphrase**, elige una segura. Es opcional pero recomendable: si te roban la clave privada, sin la passphrase es inútil.
- Para no escribir la passphrase en cada conexión, cárgala en el agente:

```bash
eval "$(ssh-agent -s)"          # arranca el agente
ssh-add ~/.ssh/id_ed25519_servidorweb
```

### 4. Copiar la clave pública al servidor

**Puerto por defecto (22):**

```bash
ssh-copy-id -i ~/.ssh/id_ed25519_servidorweb.pub usuario@servidor
```

**Puerto distinto:**

```bash
ssh-copy-id -p 2222 -i ~/.ssh/id_ed25519_servidorweb.pub usuario@servidor
```

`ssh-copy-id` añade tu clave pública a `~/.ssh/authorized_keys` del servidor con los permisos correctos. Si no está disponible, hazlo manual:

```bash
cat ~/.ssh/id_ed25519_servidorweb.pub | ssh -p 2222 usuario@servidor \
  "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

### 5. Conectarse usando esa clave

```bash
ssh -i ~/.ssh/id_ed25519_servidorweb usuario@servidor
```

Con puerto:

```bash
ssh -p 2222 -i ~/.ssh/id_ed25519_servidorweb usuario@servidor
```

### 6. Simplificar conexiones con `~/.ssh/config`

Escribir `-i`, `-p` y el usuario cada vez es tedioso. Define un alias una sola vez:

```text
# ~/.ssh/config
Host servidorweb
    HostName 203.0.113.10
    User usuario
    Port 2222
    IdentityFile ~/.ssh/id_ed25519_servidorweb
    IdentitiesOnly yes
```

Ahora basta con:

```bash
ssh servidorweb
```

`IdentitiesOnly yes` evita el error `Too many authentication failures` (ver sección 11).

### 7. Eliminar una clave creada por error (local)

```bash
rm ~/.ssh/id_ed25519_servidorweb
rm ~/.ssh/id_ed25519_servidorweb.pub
```

> ⚠️ Borrar los archivos locales **no revoca** el acceso en el servidor. La pública sigue en `authorized_keys`. Revócala también allí (sección 10).

---

## 🖥️ SERVIDOR

### 8. Archivo de claves autorizadas

Las claves públicas autorizadas para un usuario viven en:

```text
~/.ssh/authorized_keys
```

Una clave pública por línea.

### 9. Ver claves autorizadas

```bash
cat ~/.ssh/authorized_keys
```

Cada línea termina con el comentario (`-C`) que pusiste al crearla, lo que ayuda a identificar de quién es.

### 10. Revocar acceso (borrar una clave)

```bash
nano ~/.ssh/authorized_keys
```

Elimina **únicamente** la línea de la clave que quieres revocar. Guarda y cierra. El cambio es inmediato.

### 11. Identificar qué línea borrar

Desde **local**, imprime la pública que quieres revocar:

```bash
cat ~/.ssh/id_ed25519_servidorweb.pub
```

Busca esa línea exacta en `authorized_keys` del servidor y bórrala. El comentario final (`usuario@servidor`) suele bastar para localizarla.

---

## 🚨 12. Flujo correcto para revocar una clave

1. **En el servidor** → borra la línea en `authorized_keys`.
2. **En local** → borra los archivos privado y público con `rm`.

El orden importa: primero corta el acceso en el servidor; el borrado local es limpieza.

---

## 🧠 13. Validación importante (caso real)

### Error típico

```text
Too many authentication failures
```

### Causa

Tu agente SSH tiene **muchas claves cargadas**. SSH las ofrece todas una por una; el servidor corta la conexión tras varios intentos fallidos antes de llegar a la correcta.

### Solución: forzar el uso de una sola clave

Para copiar la clave:

```bash
ssh-copy-id -p 2222 -i ~/.ssh/id_ed25519_servidorweb.pub -o IdentitiesOnly=yes usuario@servidor
```

Para conectarte:

```bash
ssh -p 2222 -i ~/.ssh/id_ed25519_servidorweb -o IdentitiesOnly=yes usuario@servidor
```

`IdentitiesOnly=yes` obliga a SSH a usar **solo** la clave indicada con `-i`. Dejarlo fijo en `~/.ssh/config` evita el problema de forma permanente.

---

## 🔧 14. Diagnóstico de conexión

Cuando una conexión falla y no sabes por qué, usa modo verbose:

```bash
ssh -v  usuario@servidor    # verbose
ssh -vvv usuario@servidor    # máximo detalle (handshake, claves ofrecidas)
```

Errores frecuentes:

| Síntoma | Causa probable | Solución |
|---|---|---|
| `Permission denied (publickey)` | La pública no está en `authorized_keys`, o permisos incorrectos. | Recopiar clave; ver sección 15. |
| `Too many authentication failures` | Demasiadas claves en el agente. | `IdentitiesOnly=yes` (sección 13). |
| `Connection refused` | SSH no escucha o puerto equivocado. | Verifica puerto y que el servicio `sshd` esté activo. |
| `Host key verification failed` | Cambió la clave del host (reinstalación o posible ataque). | Verifica y borra la entrada vieja: `ssh-keygen -R servidor`. |

### 15. Permisos correctos (el servidor es estricto)

SSH **ignora** `authorized_keys` si los permisos son demasiado abiertos:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

En local, la clave privada debe ser `600`:

```bash
chmod 600 ~/.ssh/id_ed25519_servidorweb
```

---

## 🔑 16. Claves SSH para GitHub / GitLab

El mismo par sirve para Git por SSH. La pública se pega en la web del proveedor en vez de en un servidor.

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_github -C "tu-email@dominio.com"
cat ~/.ssh/id_ed25519_github.pub      # copiar y pegar en GitHub → Settings → SSH keys
```

Configura el host en `~/.ssh/config`:

```text
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_github
    IdentitiesOnly yes
```

Verifica:

```bash
ssh -T git@github.com
# → Hi <usuario>! You've successfully authenticated...
```

---

## ❗ 17. Errores que debes evitar

- ❌ Subir la clave **privada** a un repositorio (ni siquiera en privado).
- ❌ Copiar la clave **privada** al servidor.
- ❌ Reutilizar la misma clave para varios servidores o clientes.
- ❌ Dejar claves sin passphrase en equipos compartidos.
- ❌ Olvidar revocar claves de personas que dejan el equipo.

---

## 🥇 18. Regla de oro

**Una clave por servidor o entorno.** Aísla el riesgo: si una clave se compromete, revocas solo ese acceso sin tocar el resto.

```text
id_ed25519_backend_prod
id_ed25519_frontend_dev
id_ed25519_clienteX
```

---

## 📌 19. Referencia rápida

```bash
# Crear
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_<nombre> -C "usuario@servidor"

# Copiar al servidor
ssh-copy-id -p <puerto> -i ~/.ssh/id_ed25519_<nombre>.pub -o IdentitiesOnly=yes usuario@servidor

# Conectar
ssh -p <puerto> -i ~/.ssh/id_ed25519_<nombre> -o IdentitiesOnly=yes usuario@servidor

# Cargar en el agente (evita reescribir passphrase)
ssh-add ~/.ssh/id_ed25519_<nombre>

# Ver autorizadas (servidor)
cat ~/.ssh/authorized_keys

# Revocar (servidor): borrar la línea
nano ~/.ssh/authorized_keys

# Borrar local
rm ~/.ssh/id_ed25519_<nombre> ~/.ssh/id_ed25519_<nombre>.pub

# Permisos correctos
chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys

# Diagnóstico
ssh -vvv usuario@servidor
```
