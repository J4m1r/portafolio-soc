# 02 — Instalación

## Método

Instalación todo-en-uno usando el script oficial de Wazuh. Este método
despliega Manager, Indexer y Dashboard en una sola VM de forma automatizada.

## Prerequisitos

- Ubuntu Server 24.04 instalado en la VM
- Mínimo 4 GB RAM y 50 GB de disco
- Acceso root o sudo
- Conexión a internet para descarga de paquetes

## Pasos de instalación

**1. Descargar el instalador oficial**
```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
curl -sO https://packages.wazuh.com/4.7/config.yml
```

**2. Generar los archivos de configuración**
```bash
bash wazuh-install.sh --generate-config-files
```

**3. Ejecutar la instalación todo-en-uno**
```bash
bash wazuh-install.sh --all-in-one
```

El script instala y configura los tres componentes. Al finalizar muestra
las credenciales de acceso al dashboard — guardarlas en ese momento.

## Problema encontrado: disco lleno durante la instalación

Durante la instalación, el proceso se interrumpió por falta de espacio en disco.

**Diagnóstico:**
```bash
df -h
# /dev/mapper/ubuntu--vg-ubuntu--lv   24G   23G  0 remaining
```

El LVM tenía el volumen lógico en 24 GB aunque el disco virtual era de 50 GB.
Ubuntu no expande el LV automáticamente al tamaño total del disco.

**Solución:**
```bash
# Extender el volumen lógico al 100% del espacio libre disponible
lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv

# Expandir el filesystem para que reconozca el nuevo tamaño
resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
```

**Verificación:**
```bash
df -h
# /dev/mapper/ubuntu--vg-ubuntu--lv   49G   26G  22G remaining
```

Con espacio disponible, se reejecutó el instalador y completó sin errores.

## Verificación post-instalación

```bash
systemctl status wazuh-manager
systemctl status wazuh-indexer
systemctl status wazuh-dashboard
```

Los tres servicios deben mostrar `active (running)`. El dashboard queda
accesible en `https://192.168.1.66` con las credenciales generadas durante
la instalación.
