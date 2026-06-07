# 03 — Máquinas Virtuales

## cosmos — Servidor Web
| Parámetro | Detalle |
|---|---|
| Host | singularity (Proxmox) |
| IP | 192.168.1.63 |
| SO | Ubuntu 24.04 |
| RAM / CPU | 4 GB / 2 cores |
| Disco | 20 GB |
| Usuario SSH | su-jmarez |

Servidor nginx que aloja todos los sitios web del lab. Tiene agente Wazuh
instalado y es monitoreado por sentinel.

**Servicios activos:**
- syvshop.com.ar — sitio web estático con HTTPS y Cloudflare
- rulitoscake.com.ar — sitio de repostería con SSL

---

## sentinel — SIEM Wazuh
| Parámetro | Detalle |
|---|---|
| Host | singularity (Proxmox) |
| IP | 192.168.1.66 |
| SO | Ubuntu 24.04 |
| RAM / CPU | 4 GB / 2 cores |
| Disco | 50 GB (extendido vía LVM durante instalación) |
| Dashboard | https://192.168.1.66 |

VM dedicada al stack Wazuh completo: Manager, Indexer y Dashboard.
Recibe telemetría de los agentes en cosmos, CHECO y syvpc01.

---

## syvdc — Domain Controller
| Parámetro | Detalle |
|---|---|
| Host | CHECO (VirtualBox) |
| IP | 192.168.1.110 |
| SO | Windows Server 2022 |
| RAM / CPU | 8 GB / 2 cores |
| Disco | 50 GB |
| Dominio DNS | syv.domain.com |
| Dominio NetBIOS | SYV |
| Nivel funcional | Windows Server 2025 |

Domain Controller del lab de Active Directory. Gestiona autenticación,
políticas de grupo y estructura de directorio para el entorno de pruebas.

**Usuarios del dominio:**
| Usuario | Nombre | Rol |
|---|---|---|
| Administrador | — | Admin built-in |
| jmarez | Jamir Marez | Usuario de dominio |
| su-jmarez | Jamir Marez Admin | Cuenta administrativa |

**GPOs activas:**
- Default Domain Policy
- Default Domain Controllers Policy

---

## syvpc01 — Cliente Windows
| Parámetro | Detalle |
|---|---|
| Host | CHECO (VirtualBox) |
| IP | 192.168.1.70 |
| SO | Windows 10 64-bit |
| RAM / CPU | 4 GB / 2 cores |
| Disco | 50 GB |

Equipo cliente unido al dominio syv.domain.com. Usado para probar
aplicación de GPOs, login de usuarios de dominio y políticas de seguridad.

---

## WSL2 + Ollama — IA Local
| Parámetro | Detalle |
|---|---|
| Host | CHECO |
| Entorno | Ubuntu (WSL2) |
| Memoria | Dinámica (host con 32 GB RAM) |
| GPU | Nvidia 6 GB VRAM |
| API | http://127.0.0.1:11434 |
| Modelo | deepseek-r1:7b |

LLM local para análisis y triage de alertas de seguridad. La integración
planeada conecta Wazuh (sentinel) → script Python → Ollama API para
enriquecimiento automático de alertas.
