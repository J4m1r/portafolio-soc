# 03 — Agentes

## Proceso de enrolamiento

El enrolamiento se realiza desde el Dashboard de Wazuh:

1. Ir a **Agents → Deploy new agent**
2. Seleccionar el sistema operativo del endpoint
3. Ingresar la IP del manager (`192.168.1.66`)
4. Copiar el comando generado y ejecutarlo en el endpoint
5. Iniciar el servicio del agente
6. Verificar en el dashboard que el agente aparece como **Active**

---

## Agente Windows — PC Windows (192.168.1.2)

**Instalación** (ejecutar en PowerShell como administrador):
```powershell
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.7.0-1.msi `
  -OutFile wazuh-agent.msi
msiexec.exe /i wazuh-agent.msi /q WAZUH_MANAGER='192.168.1.66'
NET START WazuhSvc
```

**Cobertura de monitoreo:**

| Módulo | Qué monitorea |
|---|---|
| **Windows Event Log** | Security log: logons fallidos (4625), bloqueos (4740), logons exitosos (4624) |
| **FIM** | Cambios en rutas críticas del sistema (archivos, permisos, hashes) |
| **SCA** | Verificaciones de hardening contra benchmark CIS para Windows |
| **Vulnerability Detection** | Software instalado comparado contra base de datos de CVEs |

---

## Agente Linux — cosmos (192.168.1.63)

**Instalación:**
```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --dearmor \
  -o /usr/share/keyrings/wazuh.gpg

echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] \
  https://packages.wazuh.com/4.x/apt/ stable main" \
  | tee /etc/apt/sources.list.d/wazuh.list

apt-get update
apt-get install -y wazuh-agent

WAZUH_MANAGER='192.168.1.66' systemctl start wazuh-agent
systemctl enable wazuh-agent
```

**Cobertura de monitoreo:**

| Módulo | Qué monitorea |
|---|---|
| **Syslog** | Eventos de sistema y servicios en `/var/log/syslog` |
| **PAM / Auth** | Logons SSH fallidos, uso de sudo, autenticaciones PAM |
| **FIM** | Cambios en `/etc`, `/bin`, `/usr/bin` |
| **SCA** | Verificaciones de hardening contra benchmark CIS para Linux |

---

## Verificación de agentes activos

Desde el manager, para confirmar que los agentes están conectados:

```bash
/var/ossec/bin/agent_control -l
```

En el dashboard: **Agents → Overview** muestra el estado en tiempo real
de todos los agentes con su última conexión y nivel de alertas activas.
