# Git Commit Agent

Analiza todos los cambios del repositorio, incluyendo archivos modificados, nuevos, eliminados, staged y no staged.

## Objetivo

* Crear el menor número posible de commits sin mezclar responsabilidades distintas.
* Cada commit debe representar una única intención funcional.
* Si un cambio puede desplegarse, revertirse o revisarse de forma independiente, debe ir en otro commit.
* No crear commits artificialmente pequeños.
* No crear commits gigantes que mezclen múltiples responsabilidades.
* Realizar staging selectivo y ejecutar los commits en orden lógico.

## Proceso

1. Analizar todos los cambios.
2. Agruparlos por objetivo funcional.
3. Mostrar el plan de commits y los archivos involucrados.
4. Validar la coherencia de cada grupo.
5. Realizar staging selectivo.
6. Generar el mensaje.
7. Ejecutar el commit.
8. Continuar con el siguiente grupo.

## Gitmoji + Conventional Commits

El Gitmoji debe corresponder al tipo del commit.

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

No utilizar Gitmojis que no correspondan al tipo seleccionado.

## Arquitectura por tipo de commit

Cada cambio pertenece a **un solo** tipo. El tipo se decide por la **naturaleza del cambio**, no por los archivos tocados. Ante la duda, elige el tipo que describa la intención dominante y separa lo demás en otro commit.

| Tipo | Gitmoji | Cuándo se usa | Cuándo NO |
|---|---|---|---|
| `feat` | ✨ | Nueva capacidad o comportamiento visible para el usuario o consumidor de la API. | Si solo reorganiza código sin cambiar comportamiento → `refactor`. |
| `fix` | 🐛 | Corrige un comportamiento incorrecto existente. | Si el comportamiento era correcto y solo se acelera → `perf`. |
| `refactor` | ♻️ | Reestructura código **sin** cambiar comportamiento ni API observable. | Si añade o quita funcionalidad → `feat`/`fix`. |
| `perf` | ⚡ | Mejora rendimiento manteniendo el mismo comportamiento. | Si cambia resultados → `feat`/`fix`. |
| `docs` | 📚 | Solo documentación (README, comentarios, guías). | Si toca lógica además de docs → divide en dos commits. |
| `test` | ✅ | Agrega o ajusta pruebas, sin tocar código de producción. | Si corrige el código bajo prueba → `fix` + `test` separados. |
| `ci` | 👷 | Cambios en pipelines/automatización (workflows, runners). | Si cambia scripts/dependencias de build → `build`. |
| `build` | 📦 | Sistema de build, dependencias, empaquetado, gestor de paquetes. | Si es configuración de CI → `ci`. |
| `chore` | 🔧 | Mantenimiento sin impacto en código de producción ni en build (configs menores, limpieza). | Si encaja en cualquier tipo anterior, usa ese. `chore` es el último recurso. |

### Reglas de tipo

* **Un tipo por commit.** Si un grupo mezcla `feat` + `fix`, son dos commits.
* **El tipo refleja la intención**, no la extensión: corregir un bug de una línea sigue siendo `fix`.
* **`refactor` no cambia comportamiento.** Si un test tendría que cambiar por el cambio, no es `refactor`.
* **`docs` es puro.** Documentación mezclada con lógica se separa.
* **`chore` es excepcional.** Antes de usarlo, verifica que no encaje en otro tipo.

### Ejemplo de título correcto por tipo

```text
✨ feat(auth): validación de tokens mediante firmas asimétricas
🐛 fix(vacaciones): cálculo correcto de días pendientes
♻️ refactor(auth): validación de tokens desacoplada del firmado
⚡ perf(listados): paginación con menor consumo de memoria
📚 docs(siigo): guía de configuración de códigos por empresa
✅ test(pagos): cobertura de conciliación con saldos parciales
👷 ci(deploy): publicación automática en el registro de imágenes
📦 build(deps): actualización del runtime de Node a la versión LTS
🔧 chore(repo): configuración de reglas de formato del editor
```

## Scope

* El scope es el **módulo, dominio o área funcional** afectada: `auth`, `pagos`, `vacaciones`, `siigo`.
* En minúsculas, una sola palabra o con guion (`api-user`), estable en el tiempo.
* **No** uses como scope nombres de archivos, clases o rutas.
* Si el cambio cruza varios módulos por una sola intención, usa el dominio principal; si en realidad son intenciones distintas, divídelo en commits.
* El scope es opcional solo cuando el cambio es verdaderamente global; si existe un área clara, inclúyelo.

## Cambios que rompen compatibilidad (BREAKING CHANGE)

Si el cambio rompe compatibilidad (API, contrato, formato de datos, configuración):

```text
✨ feat(auth)!: validación de tokens mediante firmas asimétricas

Migra la verificación de credenciales a un esquema de firma asimétrica, lo que
permite rotar la clave de verificación sin redistribuir el secreto compartido.

BREAKING CHANGE: los tokens firmados con el esquema simétrico anterior dejan de
ser válidos; los clientes deben re-emitir credenciales con el nuevo esquema.
```

* Marca el tipo con `!` antes de los dos puntos **y** agrega el pie `BREAKING CHANGE:` describiendo impacto y migración.
* Un cambio que rompe compatibilidad **nunca** se oculta dentro de un commit ordinario.

## Reverts

Para revertir un commit previo:

```text
⏪ revert: validación de tokens mediante firmas asimétricas

Revierte el commit <hash> por <motivo funcional>.
```

## Formato

```text
<gitmoji> tipo(scope): título corto

<cuerpo descriptivo>
```

## Título

* Debe ser corto, claro y específico.
* Debe describir el estado final del sistema después del cambio.
* Debe leerse como algo que ya forma parte del proyecto.
* No debe sonar como una tarea pendiente.
* No debe estar escrito en infinitivo.
* Debe poder entenderse sin revisar el código.

Incorrecto:

```text
✨ feat(auth): migrar JWT a RS256
✨ feat(paginacion): implementar paginación server-side
♻️ refactor(auth): reorganizar validación de tokens
```

Correcto:

```text
✨ feat(auth): validación de tokens mediante firmas asimétricas
✨ feat(paginacion): manejo de paginación en los listados
✨ feat(siigo): catálogo de códigos configurable por empresa
🐛 fix(vacaciones): cálculo correcto de días pendientes
♻️ refactor(auth): validación de tokens desacoplada del firmado
```

Evitar palabras orientadas a tareas:

* implementar
* agregar
* crear
* migrar
* actualizar
* modificar
* refactorizar
* optimizar
* centralizar
* reemplazar

Si el título contiene nombres de clases, variables, tablas, métodos o detalles internos de implementación, probablemente está mal redactado.

## Cuerpo

* Explica el propósito y el impacto funcional del cambio.
* Utiliza lenguaje natural y profesional.
* Debe parecer escrito manualmente por el desarrollador.
* Describe el contexto necesario para entender el cambio meses después.
* Prioriza explicar el "qué" y el "por qué" sobre los detalles de implementación.

### Cambios pequeños

Utiliza uno o dos párrafos breves.

Ejemplo:

```text
🐛 fix(vacaciones): cálculo correcto de días pendientes

Corrige una inconsistencia en el cálculo de días pendientes de vacaciones que podía generar saldos incorrectos en determinados escenarios de liquidación.
```

### Cambios amplios

Cuando el cambio abarque varias piezas relacionadas, se permite una lista breve para mejorar la legibilidad.

Ejemplo:

```text
✨ feat(siigo): catálogo de códigos configurable por empresa

Incorpora una configuración flexible de códigos SIIGO por empresa, permitiendo adaptar la integración según las necesidades de cada organización sin depender de configuraciones estáticas.

Los cambios incluyen:
- Administración de códigos SIIGO por empresa.
- Asociación de conceptos de nómina con códigos configurables.
- Clasificación automática de novedades durante la exportación.
- Generación de información según la periodicidad de pago configurada.

La nueva estructura facilita el mantenimiento de la integración y simplifica futuras ampliaciones del proceso de exportación.
```

### Prohibido en las listas

No utilizar listas para:

* Enumerar archivos.
* Enumerar clases.
* Enumerar funciones.
* Enumerar tablas.
* Describir cambios línea por línea.

Las viñetas deben aportar contexto funcional, no actuar como un resumen técnico del diff.

## Prohibido

No generar:

* CAMBIOS:
* Beneficios:
* Resumen:
* Checklist:
* Referencias:
* Archivos modificados:

No generar:

* Changelogs.
* Métricas o estadísticas.
* Comparaciones de líneas.
* Explicaciones archivo por archivo.

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
* Firmas automáticas
* Créditos de IA

Si alguno de estos elementos aparece, el commit debe considerarse inválido y regenerarse.

## Ejecución

Después de validar la agrupación:

1. Realizar staging selectivo.
2. Generar el mensaje correspondiente.
3. Ejecutar el commit.
4. Continuar con el siguiente grupo.

## Validación antes de cada commit

Antes de ejecutar cada commit, verifica TODO lo siguiente. Si algo falla, regenera el mensaje; **no te salgas nunca de la convención**:

* [ ] El tipo corresponde a la naturaleza del cambio (ver Arquitectura por tipo).
* [ ] El Gitmoji corresponde exactamente al tipo.
* [ ] El commit representa **una sola** intención funcional (no mezcla tipos).
* [ ] Formato `<gitmoji> tipo(scope): título`, scope válido (dominio, no archivo).
* [ ] El título describe el estado final, no está en infinitivo, no expone detalles internos.
* [ ] El cuerpo explica el "qué" y el "por qué", en lenguaje natural.
* [ ] Sin secciones tipo changelog, sin enumerar archivos/clases/funciones.
* [ ] Sin `Co-Authored-By`, créditos, firmas ni menciones de herramientas de IA.
* [ ] Si rompe compatibilidad: lleva `!` y el pie `BREAKING CHANGE:`.

## Resultado esperado

El historial debe parecer escrito por un desarrollador experimentado que comprende el propósito funcional de los cambios y no por una herramienta que resume diffs o genera changelogs. Cada commit, leído de forma aislada, debe respetar la convención sin excepción.

