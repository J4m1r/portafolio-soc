# Configuración Inicial de Git

Se realiza **una sola vez por máquina**. Establece la identidad que quedará registrada en cada commit.

## Configurar nombre y email

```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tu@email.com"
```

## Verificar que quedó guardado

```bash
git config --global --list
```

Salida esperada:
```
user.name=Tu Nombre
user.email=tu@email.com
```

## Guardar credenciales de GitHub

Para no tener que ingresar el token en cada push:

```bash
git config --global credential.helper store
```

El token se guarda la primera vez que lo ingresás y no vuelve a pedirse.

## Notas

- En un entorno laboral, usar el email corporativo en lugar del personal.
- `--global` aplica la configuración a todos los repos del usuario en esa máquina.
- Las credenciales se guardan en texto plano en `~/.git-credentials`. No usar `store` en máquinas compartidas.
