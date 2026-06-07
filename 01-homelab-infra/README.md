# Proyecto 01 — Homelab Infrastructure

## Descripción

Diseño y administración de un entorno de virtualización personal con dos
hipervisores: Proxmox VE para servicios Linux y VirtualBox para el lab
de Windows con Active Directory. El conjunto sirve como plataforma de
práctica para proyectos de seguridad, hosting de servicios propios y
experimentación con infraestructura enterprise.

## ¿Qué demuestra este proyecto?

- Administración de hipervisores en hardware físico propio (Proxmox + VirtualBox)
- Despliegue de infraestructura Windows con Active Directory y clientes unidos al dominio
- Gestión de VMs con distintos propósitos en un entorno consolidado
- Aplicación de controles de seguridad sobre la infraestructura base
- Integración de IA local (LLM) para análisis de logs de seguridad

## Hardware

| Host | Nombre | CPU | RAM | Hipervisor |
|---|---|---|---|---|
| Servidor Proxmox | singularity | Intel i7-1165G7 @ 2.80GHz | 15 GB | Proxmox VE |
| PC de escritorio | CHECO | Intel i5-10400 @ 2.90GHz | 32 GB | VirtualBox + WSL2 |

## VMs en producción

| VM | Host | SO | RAM | Propósito |
|---|---|---|---|---|
| cosmos | singularity | Ubuntu 24.04 | 4 GB | Servidor web nginx — hosting de sitios |
| sentinel | singularity | Ubuntu 24.04 | 4 GB | SIEM — Wazuh todo-en-uno |
| syvdc | CHECO | Windows Server 2022 | 8 GB | Domain Controller — Active Directory |
| syvpc01 | CHECO | Windows 10 64-bit | 4 GB | Cliente AD — pruebas de GPO y políticas |
| WSL2 + Ollama | CHECO | Ubuntu (WSL2) | — | LLM local (deepseek) para análisis de logs |

## Contenido del proyecto

| Archivo | Descripción |
|---|---|
| [01-arquitectura.md](01-arquitectura.md) | Hardware, diagrama de red e inventario de VMs |
| [02-proxmox.md](02-proxmox.md) | Configuración del hipervisor: 2FA, backups, gestión |
| [03-vms.md](03-vms.md) | Detalle de cada VM, propósito y recursos asignados |
| [04-seguridad.md](04-seguridad.md) | Controles de seguridad aplicados y mejoras planificadas |

## Estado

Operativo — cinco entornos activos cubriendo web, SIEM, Active Directory e IA local.
