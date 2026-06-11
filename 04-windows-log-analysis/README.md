# Proyecto 04 — Análisis de Logs de Autenticación Windows

## Descripción general

Investigación de eventos de logon fallidos y bloqueos de cuentas en entornos Windows Server mediante herramientas nativas de Windows Event Log y PowerShell.

Este tipo de análisis se corresponde directamente con casos de uso SOC como detección de fuerza bruta, identificación de password spray y triage de credential stuffing.

---

## Entorno

- **SO:** Windows Server 2016 / 2019 / 2022
- **Escala:** Entorno de dominio enterprise (Active Directory)
- **Herramientas:** Windows Event Viewer, PowerShell, Usuarios y Equipos de Active Directory

---

## Event IDs analizados

| Event ID | Descripción | Relevancia SOC |
|---|---|---|
| 4625 | Intento de logon fallido | Indicador de fuerza bruta / credential stuffing |
| 4740 | Cuenta de usuario bloqueada | Umbral de logons fallidos alcanzado |
| 4767 | Cuenta de usuario desbloqueada | Seguimiento de actividad post-bloqueo |
| 4624 | Logon exitoso | Línea base / verificación post-compromiso |
| 4648 | Logon con credenciales explícitas | Indicador de movimiento lateral / pass-the-hash |

---

## Flujo de investigación

### 1. Identificar origen del bloqueo
Ante un reporte de cuenta bloqueada, el primer paso era localizar qué controlador de dominio registró el evento e identificar el equipo origen que generó los logons fallidos.

Filtro aplicado en Event Viewer:
- Log: **Security**
- Event ID: **4740**
- Cuenta afectada: nombre de usuario

### 2. Rastrear patrón de logons fallidos
Tras localizar el evento de bloqueo, se correlacionaba con eventos **4625** para determinar:
- Cuántos intentos fallidos ocurrieron antes del bloqueo
- Qué equipo o servidor era el origen (campo `Caller Computer Name`)
- Si los intentos provenían de una única fuente (fuerza bruta) o múltiples fuentes (password spray)

### 3. Determinar legitimidad
Se cruzaba el equipo origen con:
- Equipo de trabajo habitual del usuario
- Horario esperado de logon
- Comportamiento de cuentas de servicio (tareas programadas o aplicaciones con credenciales desactualizadas)

### 4. Resolución y documentación
- Si era error de credenciales (usuario olvidó contraseña / app con config desactualizada): se desbloqueó la cuenta y se notificó al usuario o responsable de la aplicación
- Si el origen era anómalo: se escaló al equipo de seguridad documentando IP/hostname y franja horaria

---

## Hallazgos clave de investigaciones reales

- La mayoría de los bloqueos provenían de **credenciales en caché desactualizadas** en unidades mapeadas o tareas programadas tras un cambio de contraseña — identificadas porque el hostname origen coincidía con el propio equipo del usuario
- Los bloqueos recurrentes desde **hostnames desconocidos** o **servidores a los que el usuario no tenía motivo de acceder** se marcaban como sospechosos
- Se identificó un patrón de logons fallidos fuera del horario laboral apuntando a cuentas de servicio — escalado como posible intento de acceso no autorizado

---

## PowerShell — Consulta de origen de bloqueo

Script para consultar eventos de bloqueo desde controladores de dominio sin abrir Event Viewer manualmente:

```powershell
# Requiere: acceso al controlador de dominio, permiso de lectura en Security log
# Ejecutar desde: estación de gestión con RSAT instalado

param(
    [Parameter(Mandatory)]
    [string]$UsuarioBloqueado,
    [string]$ControladorDominio = $env:LOGONSERVER.TrimStart('\')
)

$filtro = @{
    LogName   = 'Security'
    Id        = 4740
    StartTime = (Get-Date).AddDays(-3)
}

Get-WinEvent -ComputerName $ControladorDominio -FilterHashtable $filtro -ErrorAction Stop |
    Where-Object { $_.Properties[0].Value -eq $UsuarioBloqueado } |
    Select-Object TimeCreated,
        @{N='CuentaBloqueada'; E={ $_.Properties[0].Value }},
        @{N='EquipoOrigen';    E={ $_.Properties[1].Value }},
        @{N='ControladorDC';   E={ $_.MachineName }} |
    Format-Table -AutoSize
```

---

## Mapeo SOC

| Paso de investigación | Equivalente SOC |
|---|---|
| Filtrar Security log por Event ID | Triage de alerta desde regla SIEM |
| Identificar hostname/IP origen | Extracción de Indicador de Compromiso (IOC) |
| Correlacionar con comportamiento del usuario | Análisis de comportamiento (UEBA) |
| Escalar patrón anómalo | Creación y traspaso de incidente |

---

## Lecciones aprendidas

- Los Windows Event Logs nativos contienen suficiente detalle para reconstruir patrones de ataque de autenticación sin necesidad de un SIEM
- La identificación del equipo origen (`CallerComputerName`) es la forma más rápida de distinguir error de usuario de amenaza externa
- Las cuentas de servicio sin derechos de logon interactivo son objetivos de alto valor — monitorear el Event ID 4625 específicamente en esas cuentas reduce el ruido considerablemente
