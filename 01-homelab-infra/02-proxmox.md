# 02 — Configuración de Proxmox

## Acceso a la UI

| Parámetro | Detalle |
|---|---|
| URL | https://192.168.1.230:8006 |
| Certificado | Autofirmado — excepción configurada en el navegador |
| Autenticación | Usuario + contraseña + 2FA (TOTP) |

## Autenticación de dos factores (2FA)

2FA configurado con TOTP — código rotativo integrado en Bitwarden Premium.
Esto protege el acceso a la UI de Proxmox ante acceso no autorizado a la red local.

## Gestión de VMs

Las VMs se administran desde la UI web. Operaciones habituales:

- **Crear VM:** asignar recursos (RAM, CPU, disco), seleccionar ISO, configurar red
- **Snapshots:** punto de restauración antes de cambios críticos en una VM
- **Console:** acceso directo a la VM sin necesidad de SSH o RDP
- **Migración en caliente:** no aplica en instalación single-node

## Backups

| Parámetro | Configuración |
|---|---|
| Storage | `backup-hdd` → `/mnt/backup` (HDD Seagate 500 GB) |
| Frecuencia | Diario a las 21:00 |
| Modo | Snapshot — sin interrumpir la VM |
| Compresión | zstd |
| Retención | 3 copias |
| Notificación | Email a marez321@gmail.com en caso de fallo |

El HDD tiene APM desactivado (`apm=254` en hdparm.conf) para evitar
desgaste por head parking frecuente durante las escrituras de backup.

## Hardening aplicado

- [x] 2FA habilitado en la UI
- [x] Popup de suscripción eliminado (cosmético — no afecta funcionalidad)

## Pendiente

- [ ] Activar pve-firewall con reglas básicas (actualmente deshabilitado)
- [ ] SSH hardening en el host: deshabilitar login por contraseña, solo claves
- [ ] Configurar smartd para tests SMART automáticos en `/dev/sda` con alerta por email
- [ ] Confirmar que la IP de singularity es estática en el router
- [ ] Configurar alertas de recursos (CPU/RAM/disco)
