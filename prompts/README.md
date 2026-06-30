# Prompts reutilizables

Plantillas de prompts para asistentes de IA (Claude Code, etc.) usadas en el flujo de trabajo de estos proyectos. Copia el contenido del archivo y pásalo como instrucción al asistente.

| Archivo | Para qué sirve |
|---|---|
| [commit-completo.md](./commit-completo.md) | Agente que analiza **todos** los cambios del repo, los agrupa por intención funcional y genera varios commits coherentes (Gitmoji + Conventional Commits). |
| [commit-staging.md](./commit-staging.md) | Genera el mensaje de commit **solo** para lo que ya está en staging (`git diff --staged`) y ejecuta el commit. Versión rápida de la anterior. |
| [solicitud-cambios.md](./solicitud-cambios.md) | Plantilla para pedir una implementación: lee `CLAUDE.md`, analiza el contexto, identifica módulos afectados y aplica cambios respetando convenciones. |

## Convención de commits compartida

`commit-completo.md` y `commit-staging.md` comparten reglas:

- Formato `<gitmoji> tipo(scope): título corto` + cuerpo en lenguaje natural.
- El título describe el **estado final**, no una tarea (no infinitivos como "implementar", "agregar").
- Sin changelogs, sin enumerar archivos, sin créditos de IA ni `Co-Authored-By`.

Correspondencia Gitmoji ↔ tipo: ✨ feat · 🐛 fix · ♻️ refactor · ⚡ perf · 📚 docs · ✅ test · 👷 ci · 📦 build · 🔧 chore.
