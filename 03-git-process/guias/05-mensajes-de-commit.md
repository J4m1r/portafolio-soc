# Mensajes de Commit

Un buen mensaje de commit explica **qué cambió**, no cómo lo hizo.

## Convención: Conventional Commits

```
<tipo>: <descripción corta en imperativo>
```

## Tipos

| Tipo | Cuándo usarlo |
|------|--------------|
| `feat` | agregas funcionalidad nueva |
| `fix` | corriges un bug o error |
| `docs` | cambios en documentación |
| `refactor` | reorganizas código sin cambiar funcionalidad |
| `remove` | eliminas archivos o código |
| `chore` | tareas menores (configs, dependencias) |

## Ejemplos

```
feat: agregar sección de proyectos al README
fix: corregir enlace roto en 02-siem-wazuh
docs: documentar integración Wazuh con Ollama
remove: eliminar archivos .gitkeep innecesarios
chore: actualizar configuración inicial de git
```

## Reglas

- Descripción en **imperativo** — "agregar", no "agregué" ni "agregando"
- Máximo **72 caracteres** en la descripción
- Sin punto final
- Que explique **qué hace**, no cómo lo hace

## Commit nuevo vs amend

Siempre es mejor práctica hacer un **commit nuevo**, incluso para cambios pequeños.

| Acción | Cuándo usarla |
|--------|--------------|
| Commit nuevo | siempre, especialmente si ya hiciste `push` |
| `git commit --amend` | solo si **no** hiciste `push` y quieres corregir el último commit |

Si ya hiciste `push` y usas `amend`, el historial local y el remoto divergen y genera conflictos.

## El hash del commit

Cada commit recibe un hash único generado por git (ej: `aab5e8d`). Es inmutable — si reescribes un commit con `amend` o `rebase`, se genera un hash nuevo y el anterior deja de existir en el historial.
