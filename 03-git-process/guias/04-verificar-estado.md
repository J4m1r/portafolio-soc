# Verificar Estado del Repositorio

Comandos para inspeccionar el estado actual del repo y su historial.

## git status — estado actual

```bash
git status
```

Muestra qué archivos fueron modificados, cuáles están en staging y cuáles no están siendo rastreados.

Posibles estados de un archivo:
- **Untracked** — Git lo ve pero no lo rastrea. Solución: `git add`
- **Modified** — fue modificado desde el último commit
- **Staged** — está listo para el próximo commit
- **Nothing to commit** — todo está limpio y sincronizado

### Ejemplo de salida

```
On branch main
Changes to be committed:
        new file: README.md

Untracked files:
        01-homelab-infra/
```

## git log — historial de commits

```bash
git log
```

Muestra la lista de commits con autor, fecha y mensaje.

```bash
git log --oneline
```

Versión resumida — solo muestra el ID corto y el mensaje. Más útil para una vista rápida.

### Ejemplo de salida

```
cb053e7 agrega carpeta 03-git-process
079e84f primer commit: estructura inicial del portafolio
```

## Notas

- Usar `git status` frecuentemente, antes y después de cada operación.
- El ID del commit (ej: `079e84f`) sirve para identificar versiones específicas del historial.
