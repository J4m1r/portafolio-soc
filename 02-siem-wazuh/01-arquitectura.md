# 01 — Arquitectura del Lab

## Visión general

El stack Wazuh corre en una VM dedicada llamada **sentinel**, virtualizada sobre
Proxmox. Los agentes envían telemetría al manager desde endpoints Windows y Linux
dentro de la misma red local.

## Diagrama

```
┌─────────────────────────────────────────────────────────┐
│                     HOME LAB (LAN)                      │
│                                                         │
│  ┌───────────────────────────┐                          │
│  │  sentinel (192.168.1.66)  │                          │
│  │  Ubuntu 24.04             │                          │
│  │  ├── Wazuh Manager        │                          │
│  │  ├── Wazuh Indexer        │                          │
│  │  └── Wazuh Dashboard      │                          │
│  └────────────┬──────────────┘                          │
│               │                                         │
│       ┌───────┴────────┐                                │
│       │                │                                │
│  ┌────┴──────────┐  ┌──┴──────────────┐                 │
│  │  PC Windows   │  │  cosmos          │                 │
│  │  192.168.1.2  │  │  192.168.1.63    │                 │
│  │  Windows 11   │  │  Ubuntu 24.04    │                 │
│  │  Agente Wazuh │  │  Agente Wazuh    │                 │
│  └───────────────┘  └─────────────────┘                 │
└─────────────────────────────────────────────────────────┘
```

## Componentes de Wazuh

| Componente | Función |
|---|---|
| **Wazuh Manager** | Recibe eventos de los agentes, procesa reglas y genera alertas |
| **Wazuh Indexer** | Almacena y permite búsqueda sobre los logs (basado en OpenSearch) |
| **Wazuh Dashboard** | Interfaz web de visualización — accesible en https://192.168.1.66 |

Los tres componentes corren en la misma VM (instalación todo-en-uno), lo que es
adecuado para un lab y para entornos small-to-medium en producción.

## Especificaciones de sentinel

| Recurso | Detalle |
|---|---|
| SO | Ubuntu Server 24.04 |
| RAM asignada | 4 GB |
| Disco | 50 GB (LVM extendido durante la instalación) |
| IP | 192.168.1.66 (LAN) |
| Hipervisor | Proxmox VE sobre hardware físico |

## Decisiones de diseño

- **Todo-en-uno:** Manager, Indexer y Dashboard en una sola VM simplifica
  la administración en un lab personal sin sacrificar funcionalidad.
- **VM dedicada:** Aislar Wazuh en su propia VM permite asignarle recursos
  específicos y evitar interferencia con otros servicios del lab.
- **Red plana:** Todos los componentes están en la misma LAN (192.168.1.0/24),
  sin segmentación por ahora — la segmentación con VLANs está planificada
  como mejora futura.
