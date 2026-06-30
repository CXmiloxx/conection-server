# Solicitud de implementación de cambios

Plantilla de prompt para pedirle a un asistente de IA que implemente una funcionalidad o cambio en el proyecto. Define **dos fuentes de verdad** y separa con claridad el *qué* del *cómo*.

## Dos fuentes de verdad

| Documento | Define | Responde a |
|---|---|---|
| **`CLAUDE.md`** | Arquitectura, convenciones, patrones, reglas, stack, estructura de carpetas, comandos. | **CÓMO** se hace todo en este proyecto. |
| **README adjunto** *(si existe)* | El requerimiento concreto: qué se debe construir, alcance, criterios de aceptación. | **QUÉ** hay que hacer en esta tarea. |

`CLAUDE.md` manda sobre el *cómo*. El requerimiento manda sobre el *qué*. Si chocan, prevalece `CLAUDE.md` en lo técnico y se reporta el conflicto antes de continuar.

### De dónde sale el *qué*

El requerimiento puede llegar de dos formas. Usa **la que esté disponible**:

1. **README adjunto** — cuando se incluye, es la fuente del requerimiento.
2. **La tarea escrita en el mensaje** — si **no** se adjunta ningún README, el
   requerimiento es lo que el usuario describe directamente en el prompt.

En ambos casos el flujo es el mismo: `CLAUDE.md` define el *cómo*; el README **o**, en su ausencia, el texto de la solicitud define el *qué*. Si ni hay README ni una tarea clara en el mensaje, DETENTE y pide el requerimiento antes de tocar código.

---

## Prompt

```
Vas a implementar un cambio en este proyecto.

# 0. De dónde sacas el requerimiento (el QUÉ)
- Si se adjunta un README, ESE es el requerimiento: qué construir, alcance y
  criterios de aceptación.
- Si NO se adjunta README, el requerimiento es la tarea que el usuario describe
  directamente en este mensaje. Trátala con el mismo rigor que un README.
- Si no hay README ni una tarea clara en el mensaje, DETENTE y pide el
  requerimiento antes de escribir código.

# 1. Contexto (obligatorio antes de tocar código)
- Lee COMPLETO el archivo `CLAUDE.md`: arquitectura, convenciones, patrones,
  stack, estructura de carpetas, comandos de build/test y reglas del proyecto.
- `CLAUDE.md` es la autoridad sobre CÓMO se hace cada cosa. No inventes
  convenciones ni introduzcas patrones que no existan ya en el proyecto.
- Toma el requerimiento de la fuente del paso 0: es la autoridad sobre QUÉ hay
  que construir, su alcance y sus criterios de aceptación.

# 2. Análisis (antes de implementar)
- Identifica los módulos, archivos y capas afectados.
- Localiza código existente reutilizable: helpers, servicios, tipos, utilidades.
- Verifica que tu solución encaje en la arquitectura actual (capas, límites
  entre módulos, flujo de datos) sin romper contratos existentes.
- Si un requisito es ambiguo, contradice `CLAUDE.md` o falta información,
  DETENTE y pregunta antes de escribir código. No asumas.

# 3. Implementación
- Sigue ESTRICTAMENTE las convenciones de `CLAUDE.md`: nomenclatura, estructura
  de carpetas, estilo, manejo de errores, validación, logging, tipado.
- Reutiliza antes de crear. No dupliques lógica ya existente.
- Cambios mínimos y enfocados: toca solo lo necesario para el requerimiento.
- Mantén la coherencia con el código vecino (mismo idioma, mismos patrones).
- Si el proyecto tiene tests/linter/typecheck, ejecútalos y deja todo en verde.

# 4. Entrega
- Resumen breve de qué se implementó y por qué (el "qué" y el "por qué").
- Lista de archivos creados/modificados/eliminados con una línea por cada uno.
- Riesgos, supuestos tomados y pasos de verificación manual si aplican.
```

---

## NO debe hacer

- ❌ Empezar a programar **sin leer `CLAUDE.md`** primero.
- ❌ Inventar convenciones, librerías o patrones que el proyecto no usa.
- ❌ Duplicar lógica que ya existe en lugar de reutilizarla.
- ❌ Refactorizar, renombrar o "mejorar" código **fuera del alcance** del README.
- ❌ Cambiar dependencias, configuración o estructura global sin que el
  requerimiento lo pida.
- ❌ Asumir requisitos ambiguos: primero pregunta.
- ❌ Tocar archivos no relacionados con la tarea.
- ❌ Dejar tests, linter o typecheck en rojo.
- ❌ Entregar sin resumen ni lista de archivos afectados.

## SÍ debe hacer

- ✅ Leer `CLAUDE.md` y el README completos antes de actuar.
- ✅ Separar análisis (qué se afecta) de implementación (escribir el cambio).
- ✅ Respetar arquitectura, capas y convenciones existentes.
- ✅ Reutilizar código y mantener cambios mínimos y enfocados.
- ✅ Preguntar ante cualquier ambigüedad o conflicto.
- ✅ Verificar (tests/linter/typecheck) antes de dar por terminado.
- ✅ Cerrar con un resumen claro y la lista de archivos modificados.

---

## Cómo usar esta plantilla

1. Asegúrate de que el proyecto tenga un `CLAUDE.md` actualizado (el *cómo*).
2. Define el requerimiento (el *qué*) de una de estas formas:
   - Redacta un README de la tarea y adjúntalo, **o**
   - Describe la tarea directamente en el mensaje (sin README).
3. Pasa al asistente el bloque **Prompt** junto con el README (si lo hay) o con
   la descripción de la tarea.
4. Revisa que primero analice y pregunte; recién después, que implemente.
