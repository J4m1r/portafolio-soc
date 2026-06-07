# 04 — Seguridad de la Infraestructura

## Controles aplicados

### Autenticación
- **2FA en Proxmox UI** — TOTP configurado con Bitwarden Premium.
  Protege el acceso al hipervisor ante compromiso de contraseña.
- **Usuario dedicado en cosmos** — acceso SSH con usuario `su-jmarez`,
  no se usa root directamente.

### Disponibilidad y recuperación
- **Backups automáticos diarios** — snapshots comprimidos con retención
  de 3 copias en disco dedicado. Notificación por email ante fallo.
- **APM desactivado en disco de backups** — evita desgaste prematuro
  por head parking frecuente durante escrituras nocturnas.

### Monitoreo
- **Wazuh SIEM activo** — cosmos y CHECO con agentes conectados a sentinel.
  FIM, SCA y eventos de autenticación monitoreados en tiempo real.

---

## Superficie de ataque identificada y pendiente

| Control | Estado | Acción requerida |
|---|---|---|
| Firewall de Proxmox (pve-firewall) | Deshabilitado | Activar con reglas básicas de entrada/salida |
| SSH en singularity | Password habilitado | Deshabilitar auth por contraseña, usar solo claves |
| Monitoreo de disco (smartd) | No configurado | Tests SMART automáticos en `/dev/sda` con alerta email |
| IP de singularity | No confirmada como estática | Verificar reserva DHCP o IP fija en router |
| Alertas de recursos | No configuradas | Evaluar Netdata u otro para CPU/RAM/disco |

---

## Mejoras planificadas

### Segmentación de red (VLAN)
La red actual es plana — todos los dispositivos comparten el mismo segmento
`192.168.1.0/24`. La segmentación planificada dividiría en:

| Segmento | Dispositivos |
|---|---|
| Red doméstica | Dispositivos personales confiables |
| Lab de seguridad | singularity, sentinel, CHECO |
| IoT | Dispositivos de bajo nivel de confianza |

Requiere un switch administrable con soporte 802.1Q (pendiente adquirir).

### Hardening de Active Directory
- Crear estructura de OUs para separar equipos, usuarios y cuentas de servicio
- Aplicar GPOs de hardening basadas en benchmark CIS para Windows
- Auditoría de logon y cambios de directiva habilitada
