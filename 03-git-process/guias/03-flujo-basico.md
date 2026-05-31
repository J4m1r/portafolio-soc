# Flujo Básico de Git

El flujo de trabajo cotidiano con Git se resume en tres comandos.

## El flujo

```
Archivos locales  →  Staging  →  Historial local  →  GitHub
                  git add     git commit            git push
```

## Comandos

### 1. git add — agregar al staging

```bash
git add .                  # agrega todos los archivos modificados
git add nombre-archivo.md  # agrega un archivo específico
git add carpeta/           # agrega una carpeta específica
```

### 2. git commit — guardar una versión

```bash
git commit -m "descripción del cambio"
```

El mensaje debe describir qué cambió. Ejemplos de buenos mensajes:
- `agrega documentación de Wazuh`
- `corrige typo en README`
- `primer commit: estructura inicial`

### 3. git push — subir a GitHub

```bash
git push
```

## Flujo completo — ejemplo real

```bash
# 1. Crear o editar archivos
nano README.md

# 2. Ver qué cambió
git status

# 3. Agregar al staging
git add .

# 4. Guardar la versión
git commit -m "actualiza README con nueva sección"

# 5. Subir a GitHub
git push
```

## Notas

- `git add` no guarda nada, solo prepara los archivos.
- `git commit` guarda en local, no en GitHub todavía.
- `git push` es lo único que envía cambios a GitHub.
- Git no rastrea carpetas vacías — usar un archivo `.gitkeep` como placeholder.
