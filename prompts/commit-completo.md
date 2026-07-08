# Git Commit Agent

Analiza todos los cambios del repositorio, incluyendo archivos modificados, nuevos, eliminados, staged y no staged.

## Objetivo

* Crear el menor número posible de commits sin mezclar responsabilidades distintas.
* Cada commit debe representar una única intención funcional.
* Si un cambio puede desplegarse, revertirse o revisarse de forma independiente, debe pertenecer a otro commit.
* No crear commits artificialmente pequeños ni commits gigantes con múltiples responsabilidades.
* Realizar staging selectivo y ejecutar los commits en orden lógico.

## Flujo

1. Analizar todos los cambios.
2. Agruparlos por objetivo funcional.
3. Mostrar el plan de commits y los archivos incluidos.
4. Validar que cada grupo sea coherente.
5. Realizar staging selectivo.
6. Generar el mensaje.
7. Ejecutar el commit.
8. Continuar con el siguiente grupo.

---

# Gitmoji + Conventional Commits

El Gitmoji debe corresponder exactamente al tipo del commit.

| Tipo     | Gitmoji |
| -------- | ------- |
| feat     | ✨       |
| fix      | 🐛      |
| refactor | ♻️      |
| perf     | ⚡       |
| docs     | 📚      |
| test     | ✅       |
| ci       | 👷      |
| build    | 📦      |
| chore    | 🔧      |

No mezclar Gitmojis con categorías distintas.

---

# Selección del tipo

Cada commit debe pertenecer únicamente a uno de estos tipos:

* **feat:** nueva funcionalidad.
* **fix:** corrección de errores.
* **refactor:** reorganización interna sin cambiar comportamiento.
* **perf:** optimización de rendimiento.
* **docs:** documentación.
* **test:** pruebas.
* **ci:** integración continua.
* **build:** sistema de compilación o dependencias.
* **chore:** mantenimiento que no encaja en otra categoría.

Si un grupo mezcla varios tipos, divídelo en varios commits.

---

# Scope

El scope debe representar el módulo o dominio afectado.

Buenos ejemplos:

* auth
* clientes
* nomina
* vacaciones
* siigo
* pagos

Evita usar:

* nombres de archivos
* componentes React
* clases
* rutas
* variables

---

# Formato

```text
<gitmoji> tipo(scope): título

<cuerpo>
```

---

# Título

El título es la parte más importante del commit.

Debe responder de forma natural a la pregunta:

> **¿Qué cambió en este commit?**

Debe sentirse escrito por una persona, no por una herramienta.

Debe:

* ser corto;
* ser claro;
* describir el cambio principal;
* utilizar lenguaje natural;
* poder entenderse sin revisar el código.

No debe sonar como:

* una tarea pendiente;
* documentación técnica;
* un changelog;
* una descripción de implementación.

## Evitar

* implementar...
* agregar...
* crear...
* migrar...
* actualizar...
* modificar...
* refactorizar...
* optimizar...
* centralizar...

También evita títulos demasiado técnicos:

* breadcrumb con nombre del cliente
* JWT RS256
* PrismaService
* Redis cache
* DTO de clientes

Y evita títulos demasiado abstractos:

* navegación contextual
* experiencia mejorada
* validación segura
* arquitectura flexible

## Preferir

* encabezados con información del cliente
* manejo de paginación en los listados
* filtros para búsqueda de colaboradores
* configuración de códigos por empresa
* cálculo correcto de días pendientes
* validación de tokens mediante firmas asimétricas

## Nivel de abstracción

Describe el cambio con el mismo nivel de detalle con el que se lo explicarías a otro desarrollador.

No describas:

* el archivo;
* el componente;
* el método;
* la variable;
* la clase.

Tampoco intentes resumir una funcionalidad completa si el cambio fue mucho más pequeño.

Pregunta antes de escribir el título:

> "¿Cómo le explicaría en una frase a un compañero qué cambió?"

La respuesta normalmente será un buen título.

---

# Cuerpo

El cuerpo debe explicar:

* qué cambió;
* por qué se hizo;
* qué impacto tiene.

Debe utilizar lenguaje natural.

No debe parecer un resumen automático del diff.

## Cambios pequeños

Uno o dos párrafos son suficientes.

Ejemplo:

```text
🐛 fix(vacaciones): cálculo correcto de días pendientes

Corrige una inconsistencia en el cálculo de vacaciones que podía generar un saldo incorrecto al consultar colaboradores con múltiples periodos acumulados.
```

## Cambios grandes

Cuando el cambio abarque varias áreas relacionadas, puedes utilizar una lista breve.

Ejemplo:

```text
✨ feat(siigo): configuración de códigos por empresa

Permite administrar la configuración de códigos SIIGO de forma independiente para cada empresa, eliminando la dependencia de configuraciones estáticas.

Los cambios principales incluyen:

- Administración de códigos por empresa.
- Asociación de conceptos con códigos configurables.
- Exportación basada en la configuración de cada cliente.

La nueva estructura facilita el mantenimiento de la integración y reduce la necesidad de realizar cambios en código para soportar nuevos clientes.
```

Las listas solo deben describir funcionalidades o comportamientos.

Nunca deben enumerar:

* archivos;
* clases;
* funciones;
* tablas;
* cambios línea por línea.

---

# Prohibido

No generar:

* CAMBIOS:
* Beneficios:
* Resumen:
* Checklist:
* Referencias:
* Archivos modificados:

No generar:

* changelogs;
* listas de archivos;
* métricas;
* estadísticas;
* comparaciones;
* explicaciones archivo por archivo.

No incluir:

* Co-Authored-By
* Generated by
* Created with
* Claude
* Anthropic
* ChatGPT
* OpenAI
* Gemini
* Copilot
* firmas automáticas
* créditos de IA

Si cualquiera de estos elementos aparece, el commit debe regenerarse.

---

# Validación final

Antes de ejecutar el commit verifica que:

* El tipo es correcto.
* El Gitmoji corresponde al tipo.
* El commit representa una sola responsabilidad.
* El título suena natural.
* El título no está en infinitivo.
* El título no describe detalles internos de implementación.
* El cuerpo explica el propósito y el impacto.
* No existen secciones tipo changelog.
* No aparecen firmas ni créditos de IA.

---

# Resultado esperado

El historial debe parecer escrito por un desarrollador experimentado.

Cada commit debe explicar de forma clara **qué cambió**, utilizando un lenguaje natural, con el nivel de detalle adecuado y sin exponer detalles innecesarios de implementación.
