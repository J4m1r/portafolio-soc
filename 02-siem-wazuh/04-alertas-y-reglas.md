# 04 — Alertas y Reglas

## Escala de severidad de Wazuh

Wazuh clasifica todas las alertas en una escala del 0 al 15:

| Nivel | Clasificación | Descripción |
|---|---|---|
| 0–3 | Informativo | Actividad normal del sistema, sin relevancia de seguridad |
| 4–6 | Bajo | Eventos a registrar pero sin acción inmediata requerida |
| 7–10 | Medio | Requiere revisión — posible actividad anómala |
| 11–14 | Alto | Acción recomendada — indicador de compromiso o falla de hardening |
| 15 | Crítico | Acción inmediata requerida |

---

## Alertas observadas en el lab

### Autenticación — Linux (agente cosmos)

**Logon SSH fallido — nivel 5**
```
Rule: 5710 - sshd: Attempt to login using a non-existent user
Location: cosmos -> /var/log/auth.log
srcip: 192.168.1.8
user: usuario_inexistente
```
Generado al intentar conectarse con un usuario que no existe en el sistema.
Nivel bajo de forma individual — relevante en volumen (múltiples intentos = fuerza bruta).

**Uso de sudo — nivel 4**
```
Rule: 5402 - Successful sudo to ROOT executed
Location: cosmos -> /var/log/auth.log
user: su-jmarez
command: /usr/bin/apt
```
Actividad esperada. Útil para auditoría de qué comandos se ejecutan como root.

---

### Autenticación — Windows (agente PC Windows)

**Logon fallido — nivel 5**
```
Rule: 60122 - Windows logon failure
Location: PC-Windows -> Security
Event ID: 4625
TargetUserName: jmarez
LogonType: 3 (Network)
IpAddress: 192.168.1.8
```
Logon de red fallido. LogonType 3 indica acceso desde otro equipo de la red.

---

### File Integrity Monitoring (FIM)

**Modificación de archivo — nivel 7**
```
Rule: 550 - Integrity checksum changed
Location: cosmos -> syscheck
File: /etc/ssh/sshd_config
Size changed: yes
MD5 changed: yes
Inode changed: no
```
FIM detecta cualquier cambio en archivos monitoreados: contenido, permisos,
propietario o hash. Un cambio en `sshd_config` es de alta relevancia operativa.

---

### Security Configuration Assessment (SCA)

El módulo SCA ejecuta checks contra el benchmark CIS y reporta el resultado
de cada control. Ejemplo de findings en el agente Windows:

| Check | Resultado | Descripción |
|---|---|---|
| Ensure Windows Firewall is enabled | **FAIL** | Firewall deshabilitado en perfil público |
| Ensure guest account is disabled | PASS | Cuenta guest deshabilitada |
| Ensure audit logon events is enabled | PASS | Auditoría de logon activa |
| Ensure screen lock timeout is configured | **FAIL** | Sin timeout de pantalla configurado |

Los findings FAIL representan superficies de ataque concretas y son base
para tareas de hardening.

---

## Triage de una alerta — flujo básico

Ante una alerta de nivel 7 o superior el flujo aplicado en el lab es:

1. **Identificar** — ¿qué regla disparó? ¿qué agente? ¿qué usuario/IP?
2. **Contextualizar** — ¿es actividad esperada en ese horario y desde ese origen?
3. **Correlacionar** — ¿hay otros eventos relacionados en la misma ventana de tiempo?
4. **Decidir** — falso positivo, actividad legítima a documentar, o escalada

---

## Pendiente en desarrollo

- Reglas personalizadas para umbrales de fuerza bruta (5+ intentos en 60 segundos)
- Regla para logon fuera de horario laboral
- Pipeline de notificaciones por email para alertas nivel 12+
- Queries Lucene para threat hunting en el dashboard
