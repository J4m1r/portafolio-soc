# 05 — Integración Wazuh → Ollama (AI Triage)

## Objetivo

Enriquecer automáticamente las alertas de Wazuh con análisis de IA local, sin depender de servicios externos ni incurrir en costos por token. Cada alerta crítica es analizada por un LLM que explica qué ocurrió, evalúa la criticidad y recomienda una acción.

---

## Arquitectura del pipeline

```
Agente (syvdc/cosmos)
        ↓
Wazuh Manager (sentinel — 192.168.1.66)
        ↓ alerta nivel 7+
Custom Integration (ossec.conf)
        ↓
Script Python (/var/ossec/integrations/custom-ollama)
        ↓ HTTP POST
Ollama API (nebula — 192.168.1.2:11434)
        ↓ deepseek-r1:7b
Log enriquecido (/var/ossec/logs/ollama-analysis.log)
```

---

## Componentes

| Componente | Ubicación | Función |
|---|---|---|
| `custom-ollama` | `/var/ossec/integrations/` | Script Python que conecta Wazuh con Ollama |
| `ossec.conf` | `/var/ossec/etc/` | Define cuándo y cómo disparar la integration |
| `ollama-analysis.log` | `/var/ossec/logs/` | Log con cada alerta + análisis del modelo |
| Ollama + deepseek-r1:7b | nebula (WSL2, 192.168.1.2:11434) | Motor de inferencia local |

---

## Nebula — specs de la máquina LLM

| Parámetro | Valor |
|---|---|
| Host | CHECO (192.168.1.2) — PC Windows |
| Entorno | WSL2 Ubuntu 22.04 |
| RAM asignada | 8 GB (.wslconfig) |
| CPUs asignados | 2 |
| GPU | NVIDIA RTX 2060 — 6144 MB VRAM |
| Modelo | deepseek-r1:7b (4.7 GB en VRAM) |
| Endpoint | http://192.168.1.2:11434 |

El modelo corre completamente en VRAM — no consume RAM del sistema durante inference.

---

## Configuración en ossec.conf

Bloque agregado al final de `/var/ossec/etc/ossec.conf`:

```xml
<integration>
  <name>custom-ollama</name>
  <level>7</level>
  <alert_format>json</alert_format>
</integration>
```

- `name`: nombre del script en `/var/ossec/integrations/`
- `level`: nivel mínimo de alerta para disparar la integration
- `alert_format`: formato en que Wazuh pasa la alerta al script

Wazuh tiene integrations nativas (Slack, PagerDuty, VirusTotal). Esta es una **custom integration** — mismo mecanismo, script propio.

Cada vez que se modifica `ossec.conf` hay que reiniciar el manager:

```bash
systemctl restart wazuh-manager
```

---

## Script: custom-ollama

**Ruta:** `/var/ossec/integrations/custom-ollama`  
**Permisos:** `750` — owner `root:wazuh`

```python
#!/usr/bin/env python3
import sys
import json
import requests
from datetime import datetime

OLLAMA_URL = "http://192.168.1.2:11434/api/generate"
MODEL = "deepseek-r1:7b"
LOG_FILE = "/var/ossec/logs/ollama-analysis.log"

def analyze_alert(alert):
    prompt = f"""Eres un analista SOC. Analiza esta alerta de Wazuh y responde en español:
1. ¿Qué ocurrió?
2. ¿Es crítico? ¿Por qué?
3. Acción recomendada

Alerta:
{json.dumps(alert, indent=2, ensure_ascii=False)}"""

    response = requests.post(OLLAMA_URL, json={
        "model": MODEL,
        "prompt": prompt,
        "stream": False
    }, timeout=120)

    return response.json().get("response", "Sin respuesta")

def main():
    alert_file = sys.argv[1]
    with open(alert_file) as f:
        alert = json.load(f)

    analysis = analyze_alert(alert)

    entry = {
        "timestamp": datetime.utcnow().isoformat(),
        "alert_id": alert.get("id", "unknown"),
        "rule": alert.get("rule", {}),
        "agent": alert.get("agent", {}),
        "analysis": analysis
    }

    with open(LOG_FILE, "a") as f:
        f.write(json.dumps(entry, ensure_ascii=False) + "\n")

if __name__ == "__main__":
    main()
```

---

## Instalación paso a paso

**1. Verificar conectividad sentinel → nebula:**
```bash
curl http://192.168.1.2:11434/api/tags
```
Debe devolver el modelo `deepseek-r1:7b` en la lista.

**2. Instalar dependencia Python:**
```bash
sudo apt install python3-requests -y
```

**3. Crear el script:**
```bash
nano /var/ossec/integrations/custom-ollama
# pegar el contenido del script
```

**4. Asignar permisos:**
```bash
chmod 750 /var/ossec/integrations/custom-ollama
chown root:wazuh /var/ossec/integrations/custom-ollama
```

**5. Agregar integration en ossec.conf:**
```bash
nano /var/ossec/etc/ossec.conf
# agregar el bloque <integration> antes del </ossec_config> final
```

**6. Reiniciar Wazuh:**
```bash
systemctl restart wazuh-manager
```

---

## Prueba manual

Crear alerta de prueba:

```bash
echo '{"id":"test-001",' > /tmp/test-alert.json
echo '"rule":{"id":"5710","level":7,"description":"SSH brute force"},' >> /tmp/test-alert.json
echo '"agent":{"id":"001","name":"syvdc"},' >> /tmp/test-alert.json
echo '"data":{"srcip":"192.168.1.99","dstuser":"root"}}' >> /tmp/test-alert.json
```

Ejecutar el script directamente:
```bash
python3 /var/ossec/integrations/custom-ollama /tmp/test-alert.json
```

Verificar resultado:
```bash
cat /var/ossec/logs/ollama-analysis.log
```

---

## Ejemplo de salida

```json
{
  "timestamp": "2026-06-09T00:04:27.254618",
  "alert_id": "test-001",
  "rule": {
    "id": "5710",
    "level": 7,
    "description": "SSH brute force"
  },
  "agent": {
    "id": "001",
    "name": "syvdc"
  },
  "analysis": "Se detectó un ataque de fuerza bruta SSH sobre el usuario root desde 192.168.1.99. Es crítico porque root tiene los privilegios más elevados del sistema. Acción recomendada: bloquear la IP origen y revisar logs de autenticación."
}
```

---

## Monitoreo en tiempo real

```bash
tail -f /var/ossec/logs/ollama-analysis.log
```

Ver alertas nivel 7+ llegando al manager:
```bash
tail -f /var/ossec/logs/alerts/alerts.json | grep '"level":.[789]'
```

---

## Por qué deepseek-r1

| Modelo | VRAM | Razonamiento | Cybersec | Velocidad |
|---|---|---|---|---|
| deepseek-r1:7b | 4.7 GB | ⭐⭐⭐⭐⭐ (CoT) | ⭐⭐⭐⭐ | GPU — rápida |
| llama3.1:8b | 5.0 GB | ⭐⭐⭐ | ⭐⭐⭐ | GPU — rápida |
| deepseek-r1:14b | 9.0 GB | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | CPU — lenta |

El sufijo `r1` indica razonamiento paso a paso (Chain of Thought) — el modelo "piensa" antes de responder, lo que mejora significativamente la calidad del análisis de seguridad comparado con modelos de mismo tamaño sin razonamiento.

---

## Pendiente (Fase 2 y 3)

- [ ] Integrar OpenClaw para notificaciones automáticas por Telegram u otro canal
- [ ] Mostrar análisis de Ollama en el dashboard de Wazuh como alerta enriquecida
