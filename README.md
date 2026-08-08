# 🌐 TapMap Docker - Monitor de conexiones de red en tiempo real con mapa interactivo

[![GitHub](https://img.shields.io/badge/GitHub-Repo-blue?logo=github)](https://github.com/olalie/tapmap)
[![Docker](https://img.shields.io/badge/Docker-olalie%2Ftapmap-blue?logo=docker)](https://hub.docker.com/r/olalie/tapmap)
[![License](https://img.shields.io/badge/License-MIT-green)](https://opensource.org/licenses/MIT)

## 📋 Descripción general

**TapMap** es una herramienta de conciencia de red que visualiza en tiempo real en un mapa interactivo dónde conecta tu ordenador. Inspecciona datos de sockets locales, enriquece direcciones IP con geolocalización MaxMind, y muestra todas las conexiones activas en un mapa global. Es una herramienta de *awareness*, no un firewall o suite de seguridad completa, pero hace visible algo que generalmente está oculto: dónde están realmente los servidores con los que tu ordenador habla.

Arquitectura simple: Python (Dash + Plotly) que lee sockets locales (`netstat`/procfs), consulta bases de datos locales de geolocalización MaxMind, y renderiza mapa interactivo. **Cero telemetría. Datos 100% locales.** Compatible con Windows, macOS, Linux y Docker.

## ✨ Características principales

- 🗺️ **Mapa interactivo global** - Todas las conexiones activas visualizadas en mapa con zoom, pan y marcadores por ubicación
- 📍 **Geolocalización MaxMind** - Bases de datos locales `.mmdb` para ubicación exacta de cada IP sin cloud lookups
- 📊 **Historial 30 días** - Rolling history con insights sobre patrones, conexiones inusuales y actividad diaria
- 🔄 **Detección de cambios** - Nueva ubicación (amarillo), ubicación lejana (rojo), conocida (verde) con visual patterns
- ⚙️ **Información de proceso** - Qué app/servicio está conectando (si permisos lo permiten)
- 📈 **Daily Report & Insights** - Análisis de patrones, qué cambió, actividad por día de la semana
- 🔒 **100% local** - Todos los datos en tu máquina, cero cloud, cero telemetría
- ⚙️ **Configurable** - Puerto, ubicación local (manual o auto), filtros, base de datos GeoIP
- 💻 **Multi-plataforma** - Windows, macOS, Linux, Docker. Ejecutable portable o contenedor
- 📦 **Free & Open Source** - Código abierto, MIT licensed

## 📋 Requisitos del sistema

- **Docker** en Linux (requerido para Docker; Windows/macOS necesita app nativa)
- **256 MB - 1 GB RAM** (muy ligero)
- **500 MB - 2 GB espacio disco** (depende datos históricos + GeoIP DB)
- **Puerto 8080** (configurable con `TAPMAP_PORT`)
- **Bases de datos MaxMind GeoLite2** (`.mmdb` files descargados aparte)
- **Capacidades Linux**: `SYS_PTRACE` para visibilidad de procesos (opcional)
- **Navegador moderno** para UI web
- **GeoIP databases**: Descargar gratis desde MaxMind (requiere registro), actualizar mensualmente recomendado. Formato: `GeoLite2-City.mmdb`

## 🐳 Instalación

### Opción 1: Docker CLI

```bash
# Paso 1: Crear estructura de carpetas
mkdir -p ~/tapmap-data/geoip
cd ~/tapmap-data

# Paso 2: Descargar GeoIP databases
# Ir a https://www.maxmind.com/en/account
# Crear cuenta (gratis, requiere email)
# Download GeoLite2-City.mmdb (formato .mmdb)
# Copiar a ~/tapmap-data/geoip/GeoLite2-City.mmdb

# Paso 3: Ejecutar TapMap en Docker
docker run --rm \
  --network host \
  --pid host \
  -v ~/tapmap-data:/data \
  -v ~/tapmap-data/geoip:/data/geoip \
  -e TAPMAP_IN_DOCKER=1 \
  -e TAPMAP_PORT=8080 \
  olalie/tapmap:latest
```

### Opción 2: Docker Compose

```yaml
version: '3.8'

services:
  tapmap:
    image: olalie/tapmap:latest
    container_name: tapmap
    network_mode: host
    pid: host
    environment:
      - TAPMAP_IN_DOCKER=1
      - TAPMAP_PORT=8080
      # Configuración de ubicación local (elige una):
      # -e TAPMAP_LON=auto -e TAPMAP_LAT=auto    # Opción 1: Auto (detecta IP pública, lento)
      # -e TAPMAP_LON=10.7522 -e TAPMAP_LAT=59.9139  # Opción 2: Manual (coordenadas fijas, rápido)
      # -e TAPMAP_LON=none                        # Opción 3: Deshabilitado (sin marcador local)
    volumes:
      - ~/tapmap-data:/data
      - ~/tapmap-data/geoip:/data/geoip
    restart: unless-stopped
    # Para visibilidad de procesos en Linux (opcional):
    # cap_add:
    #   - SYS_PTRACE
```

```bash
# Ejecutar con Docker Compose
docker compose up -d
```

## ⚙️ Configuración

1. **Variables de entorno principales**:
   - `TAPMAP_IN_DOCKER=1` - Obligatorio para ejecución en contenedor
   - `TAPMAP_PORT=8080` - Puerto de la interfaz web (configurable)

2. **Configuración de ubicación local** (elige una):
   - **Auto**: `TAPMAP_LON=auto` + `TAPMAP_LAT=auto` (detecta IP pública, más lento)
   - **Manual**: `TAPMAP_LON=<longitud>` + `TAPMAP_LAT=<latitud>` (coordenadas fijas, rápido)
   - **Deshabilitado**: `TAPMAP_LON=none` (sin marcador local en el mapa)

3. **Volúmenes requeridos**:
   - `~/tapmap-data:/data` - Datos persistentes (historial, configuración)
   - `~/tapmap-data/geoip:/data/geoip` - Bases de datos MaxMind `.mmdb`

4. **Capacidades Linux opcionales**:
   - `SYS_PTRACE` - Para visibilidad de procesos (requiere `--pid host`)

5. **Red y PID**:
   - `network_mode: host` - Necesario para ver conexiones de red del host
   - `pid: host` - Necesario para inspección de procesos

## 🚀 Primeros pasos

1. **Acceder al dashboard**
   - Abre `http://localhost:8080` en tu navegador
   - Espera 5-10 segundos para que aparezcan las conexiones
   - El mapa muestra todas las IPs a las que conectas

2. **Interpretar el mapa**
   - 🟢 **Verde**: Ubicaciones conocidas/frecuentes
   - 🟡 **Amarillo**: Nueva ubicación (vista recientemente)
   - 🔴 **Rojo**: Ubicación lejana (lejos de tu posición)
   - Click en marcador → Ver IPs, puertos, procesos conectados

3. **Revisar Insights**
   - Pestaña "Insights" muestra patrones históricos
   - Actividad por hora del día
   - Cambios recientes (nuevas ubicaciones, conexiones)
   - Conexiones inusuales flaggeadas

4. **Revisar Daily Activity Report**
   - Pestaña "Daily Activity" muestra historial 30 días
   - Actividad por fecha
   - Patrones sobre tiempo

5. **Ver información de proceso**
   - Click en ubicación en el mapa
   - Expandir "Connections" para ver qué app conecta
   - Proceso, IP remota, puerto local/remoto

6. **Configurar alertas/custom**
   - Settings → personaliza ubicación local, puerto, comportamiento de insights

## 💡 Casos de uso

- 🔍 **Detección de malware**: Conexiones inusuales = posible malware/ransomware
- 📋 **Auditoría de servicios**: Qué servicios de background conectan dónde
- 🛡️ **Privacy monitoring**: Detecta qué apps conectan con servidores remotos
- 🔧 **Network troubleshooting**: Ver conexiones activas, diagnóstico de issues
- 🎯 **Security awareness**: Entiende tu "footprint" de red
- 🚨 **Incident response**: Investigar compromisos, detectar exfiltración
- 📚 **Learning**: Educación sobre cómo funciona internet

## 🔒 Acceso remoto seguro

> **⚠️ Importante**: TapMap expone información sensible de red. No lo expongas directamente a internet.

Para acceso remoto seguro, usa:
- **VPN** (WireGuard, Tailscale, OpenVPN)
- **Reverse Proxy** con autenticación (Authelia, Authentik, Nginx Proxy Manager + Basic Auth)
- **SSH Tunnel**: `ssh -L 8080:localhost:8080 user@server`

## 🛠️ Gestión y mantenimiento

### Ver logs de TapMap
```bash
docker logs -f tapmap
# O si usas docker run:
docker logs -f <container_id>
```

### Actualizar GeoIP databases (mensualmente recomendado)
```bash
# 1. Descargar nuevo GeoLite2-City.mmdb desde MaxMind
# 2. Copiar a ~/tapmap-data/geoip/
# 3. En UI: click "Recheck GeoIP databases"
cp ~/Downloads/GeoLite2-City.mmdb ~/tapmap-data/geoip/
```

### Backup de historial
```bash
cp -r ~/tapmap-data /backup/tapmap-data-$(date +%Y%m%d)
```

### Reiniciar TapMap
```bash
docker restart tapmap
# O detener y reiniciar el comando docker run
```

### Limpiar datos históricos antiguos
TapMap mantiene 30 días rolling. Automáticamente limpia datos más antiguos. No requiere mantenimiento manual.

## 📝 Licencia

Este proyecto utiliza la imagen Docker de **olalie/tapmap**. El código original de TapMap es open source bajo licencia MIT (aunque upstream puede variar). Consulta el [repositorio oficial](https://github.com/olalie/tapmap) para detalles de licencia actualizados.

---

> 📖 **Basado en el tutorial**: [Cómo instalar TapMap en Docker - Monitor de conexiones de red en tiempo real con mapa interactivo en Docker](https://genbyte.blogspot.com/2026/07/como-instalar-tapmap-en-docker-monitor.html)