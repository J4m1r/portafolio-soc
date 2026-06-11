# Sincronizar Repositorio

Cuando hay cambios en GitHub que no tienes en local, necesitas sincronizar.

## Verificar si hay cambios remotos

```bash
git fetch origin
git log --oneline origin/main -5
```

`git fetch` descarga la información de GitHub sin modificar tus archivos. Luego puedes comparar el historial remoto con el local.

## Traer los cambios a local

```bash
git pull origin main
```

`git pull` hace dos cosas en una:

```
GitHub  →  repo local  →  archivos en disco
        git fetch      git merge
```

1. **fetch** — descarga los commits nuevos de GitHub
2. **merge** — integra esos commits y actualiza los archivos en disco

## Ver qué archivos cambiaron

```bash
git diff <hash-anterior> <hash-nuevo> --name-status
```

Ejemplo:

```bash
git diff fe3736e aab5e8d --name-status
```

Muestra los archivos con su estado:
- `A` — agregado
- `M` — modificado
- `D` — eliminado

## Flujo completo — ejemplo real

```bash
# 1. Ver si el repo está actualizado
git status

# 2. Descargar info de GitHub sin modificar archivos
git fetch origin

# 3. Comparar historial remoto vs local
git log --oneline origin/main -5

# 4. Traer los cambios y actualizar archivos
git pull origin main
```

## Notas

- No hace falta re-clonar el repositorio para sincronizarlo — `git pull` es suficiente.
- Si el working tree está limpio (`nothing to commit`), el pull no genera conflictos.
- Si tienes cambios locales sin commitear, guárdalos con `git stash` antes de hacer pull.
