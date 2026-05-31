# Clonar un Repositorio

Clonar significa descargar una copia completa de un repo remoto (GitHub) a tu máquina local, incluyendo todo el historial de commits.

## Comando

```bash
git clone https://github.com/USUARIO/nombre-repo
```

Git crea automáticamente una carpeta con el nombre del repo en el directorio donde estés parado.

## Ejemplo real

```bash
cd ~/portafolio-local
git clone https://github.com/J4m1r/portafolio-soc
```

Resultado: se crea la carpeta `portafolio-soc/` lista para trabajar.

## Verificar que el clone fue exitoso

```bash
cd nombre-repo
git status
```

Salida esperada:
```
On branch main
Your branch is up to date with 'origin/main'.
nothing to commit, working tree clean
```

## Notas

- Si el repo está vacío, Git avisa pero el clone es válido.
- Solo se clona una vez. Para actualizar un repo ya clonado se usa `git pull`.
- `origin` es el nombre que Git le da automáticamente al repo remoto de origen.
