# Proyecto 02 — Lab SIEM con Wazuh

## Descripción

Despliegue y operación de un stack Wazuh completo en entorno de home lab personal.
El proyecto cubre la instalación del servidor, el enrolamiento de agentes en
plataformas mixtas (Windows y Linux) y el análisis de alertas en tiempo real.

## ¿Qué demuestra este proyecto?

- Capacidad de desplegar y mantener un SIEM en entorno real, no solo teórico
- Manejo de endpoints mixtos (Windows + Linux) como en infraestructura enterprise
- Interpretación de alertas por severidad y correlación con eventos de seguridad
- Troubleshooting de infraestructura durante la implementación

## Infraestructura

| Componente | VM / Host | IP | SO |
|---|---|---|---|
| Wazuh Manager + Dashboard + Indexer | sentinel | 192.168.1.66 | Ubuntu 24.04 |
| Agente Windows | PC Windows | 192.168.1.2 | Windows 11 |
| Agente Linux | cosmos | 192.168.1.63 | Ubuntu 24.04 |

## Contenido del proyecto

| Archivo | Descripción |
|---|---|
| [01-arquitectura.md](01-arquitectura.md) | Infraestructura del lab, componentes y diagrama |
| [02-instalacion.md](02-instalacion.md) | Proceso de instalación y problemas resueltos |
| [03-agentes.md](03-agentes.md) | Configuración de agentes Windows y Linux |
| [04-alertas-y-reglas.md](04-alertas-y-reglas.md) | Alertas observadas, niveles de severidad y análisis |
| [05-scripts.md](05-scripts.md) | Scripts usados para gestión y monitoreo |

## Estado

Activo — monitoreo operativo con agentes Windows y Linux conectados.
