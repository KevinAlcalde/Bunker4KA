# Bunker 4 — Plataforma de Automatizacion e Infraestructura Privada

Infraestructura self-hosted sobre **Proxmox VE**, accesible desde internet mediante un bastion SSH y Nginx Proxy Manager. Corre un stack Docker completo de automatizacion, mensajeria y monitoreo dentro de un contenedor LXC Ubuntu 24.04.

---

## Tabla de contenidos

- [Arquitectura](#arquitectura)
- [Stack de servicios](#stack-de-servicios)
- [Acceso SSH](#acceso-ssh)
- [Tuneles locales](#tuneles-locales)
- [Despliegue del stack](#despliegue-del-stack)
- [Ollama - LLM local](#ollama---llm-local)
- [Grafana](#grafana)
- [Variables de entorno](#variables-de-entorno)
- [Comandos utiles](#comandos-utiles)
- [Mantenimiento](#mantenimiento)
- [Resolucion de problemas](#resolucion-de-problemas)

---

## Arquitectura

Capas de red, de afuera hacia adentro:

```
Internet
    |
    v
[1] Modem Movistar (NAT publico)
    Puerto 10224  -->  Router MikroTik
    Puertos 80/443 ->  Router MikroTik
    |
    v
[2] Router MikroTik
    Puerto 10224  -->  Bastion (10.10.150.8)
    Puertos 80/443 ->  Nginx Proxy Manager
    |
    v   Proxmox VE  |  AMD Ryzen 7 5700G  |  ZFS pool "Tanque03"
    |
    +--[CT 103] archlinux-bastion    10.10.150.8/24  |  512 MB RAM  |  7.8 GB LVM
    |           Arch Linux, usuario kevin sin privilegios
    |           Unico punto de entrada SSH a la red interna
    |
    +--[CT]     nginx-proxy-manager  10.10.150.x/24
    |           Proxy reverso HTTPS para *.perfeccion.ar
    |
    +--[CT 108] kevin01              10.10.153.4/24  |  2 GB RAM  |  20 GB ZFS
                Ubuntu 24.04 LTS, Docker Engine
                Corre el stack completo de servicios (ver tabla abajo)
```

---

## Stack de servicios

| Servicio        | Imagen                              | Puerto interno | Puerto host | Estado   | Funcion                      |
|-----------------|-------------------------------------|---------------:|------------:|----------|------------------------------|
| Evolution API   | `evoapicloud/evolution-api:latest`  | 8080           | 8080        | Activo   | API WhatsApp (idioma: es-AR) |
| n8n             | `n8nio/n8n:latest`                  | 5678           | 5678        | Activo   | Automatizacion low-code      |
| Grafana         | `grafana/grafana:latest`            | 3000           | 3000        | Activo   | Dashboards y monitoreo       |
| Portainer CE    | `portainer/portainer-ce`            | 9000           | 9000        | Activo   | UI de gestion Docker         |
| PostgreSQL      | `postgres:14.13`                    | 5432           | 5432        | Activo   | Base de datos principal      |
| Redis           | `redis:latest`                      | 6379           | -           | Activo   | Cache y cola de mensajes     |
| Ollama          | servicio systemd                    | 11434          | localhost   | Activo   | Inferencia LLM local         |
| Apache2         | servicio systemd                    | 80             | 80          | Activo   | Proxy reverso hacia Grafana  |
| Node-RED        | `nodered/node-red:latest`           | 1880           | -           | Detenido | Flujos IoT (pausado)         |

**Red Docker interna:** `red_gral` (bridge externo, crear con `docker network create red_gral`)

---

## Acceso SSH

### Conectar al bastion

```bash
ssh -p 10224 kevin@bunker4.perfeccion.ar
```

El bastion corre Arch Linux con usuario `kevin` sin privilegios.
Actua como unico punto de entrada SSH a la red interna.

### Saltar a kevin01 (nodo principal)

Desde el bastion:

```bash
ssh kevin@10.10.153.4
```

Directamente desde tu maquina local con ProxyJump:

```bash
ssh -J kevin@bunker4.perfeccion.ar:10224 kevin@10.10.153.4
```

### Configuracion para VSCode Remote SSH

Agregar en `~/.ssh/config`:

```
Host bunker4-bastion
    HostName bunker4.perfeccion.ar
    Port 10224
    User kevin

Host kevin01
    HostName 10.10.153.4
    User kevin
    ProxyJump bunker4-bastion
```

Conectar luego a `kevin01` desde la extension Remote SSH de VSCode.

---

## Tuneles locales

Para acceder a los servicios sin exponer puertos publicos, usar SSH port-forwarding:

```bash
# Sintaxis general
ssh -N -p 10224 \
    -L 127.0.0.1:<puerto_local>:10.10.153.4:<puerto_remoto> \
    kevin@bunker4.perfeccion.ar
```

| Servicio      | Puerto local | Destino en kevin01    | URL local                |
|---------------|-------------:|-----------------------|--------------------------|
| n8n           | 50001        | 10.10.153.4:5678      | http://localhost:50001   |
| Evolution API | 50002        | 10.10.153.4:8080      | http://localhost:50002   |
| Portainer     | 50003        | 10.10.153.4:9000      | http://localhost:50003   |
| Grafana       | 50004        | 10.10.153.4:3000      | http://localhost:50004   |
| PostgreSQL    | 50005        | 10.10.153.4:5432      | psql en localhost:50005  |
| Ollama API    | 50006        | 10.10.153.4:11434     | http://localhost:50006   |

Ejemplo con multiples forwards en simultaneo:

```bash
ssh -N -p 10224 \
    -L 127.0.0.1:50001:10.10.153.4:5678 \
    -L 127.0.0.1:50003:10.10.153.4:9000 \
    kevin@bunker4.perfeccion.ar
```

### Alternativas al tunel SSH

| Metodo      | Ventaja                              | Consideracion                               |
|-------------|--------------------------------------|---------------------------------------------|
| DevTunnels  | Integrado en VSCode, estable         | Requiere cuenta Microsoft                   |
| Cloudflared | Dominio fijo con `*.perfeccion.ar`   | Requiere YAML de configuracion              |
| ngrok       | Facil de iniciar                     | Dominio cambia en cada reinicio (plan free) |

> n8n usa actualmente Microsoft DevTunnels como webhook URL: `https://kp3pxp3p-5678.usw3.devtunnels.ms/`

---

## Despliegue del stack

### Prerrequisitos

| Componente              | Version minima |
|-------------------------|----------------|
| Ubuntu 22.04+ (LXC/VM)  | -              |
| Docker Engine           | 24.x           |
| Docker Compose Plugin   | v2.27+         |
| Acceso SSH al bastion   | Puerto 10224   |

### Pasos

```bash
# 1. Clonar el repositorio en kevin01
git clone https://github.com/KevinAlcalde/Bunker4KA.git
cd Bunker4KA

# 2. Crear la red Docker externa (primera vez)
docker network create red_gral

# 3. Configurar variables de entorno
cp .env.example .env
nano .env

# 4. Levantar el stack
docker compose -f evolutionAPI.yml up -d

# 5. Verificar estado
docker compose -f evolutionAPI.yml ps
```

---

## Ollama - LLM local

Ollama corre como servicio systemd en kevin01, escuchando en `127.0.0.1:11434`.
No esta expuesto externamente.

### Modelos instalados

| Modelo           | Tamano | Uso recomendado              |
|------------------|--------|------------------------------|
| `mistral:latest` | 4.4 GB | Chat, razonamiento general   |
| `qwen2.5:0.5b`   | 397 MB | Tareas livianas, embeddings  |

### Comandos de Ollama

```bash
# Listar modelos disponibles
ollama list

# Chat interactivo
ollama run mistral

# Llamada REST (desde kevin01)
curl http://localhost:11434/api/generate \
  -d '{"model":"mistral","prompt":"Hola","stream":false}'

# Agregar un modelo nuevo
ollama pull llama3.2
```

Para acceder a Ollama desde la maquina local:

```bash
ssh -N -p 10224 \
    -L 127.0.0.1:11434:10.10.153.4:11434 \
    kevin@bunker4.perfeccion.ar
```

---

## Grafana

Grafana corre en el puerto `3000` y esta expuesto via Apache2 como proxy reverso.

- **URL publica:** http://grafana.kevin.perfeccion.ar
- **Usuario administrador:** `grafkev`

Configuracion de Apache en `/etc/apache2/sites-enabled/`:

```apache
<VirtualHost *:80>
    ServerName grafana.kevin.perfeccion.ar
    ProxyPreserveHost On
    ProxyPass        / http://localhost:3000/
    ProxyPassReverse / http://localhost:3000/
    RequestHeader set X-Forwarded-Proto http
    RequestHeader set X-Forwarded-Host grafana.kevin.perfeccion.ar
</VirtualHost>
```

---

## Variables de entorno

Crear el archivo `.env` a partir del ejemplo:

```bash
cp .env.example .env
```

Variables principales:

```dotenv
# Servidor
SERVER_PORT=8080
TZ=America/Argentina/Buenos_Aires

# Evolution API
AUTHENTICATION_API_KEY=<clave_hex_32>

# PostgreSQL
POSTGRES_USER=postgres
POSTGRES_PASSWORD=<contrasena_segura>
POSTGRES_DB=evolution
```

Generar una API key segura:

```bash
openssl rand -hex 32
```

---

## Comandos utiles

```bash
# Estado de todos los contenedores
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# Logs en tiempo real
docker logs -f evolution_api
docker logs -f n8n

# Entrar al shell de un contenedor
docker exec -it evolution_api bash
docker exec -it n8n sh

# Reiniciar un servicio
docker restart grafana
docker restart n8n

# Recursos en tiempo real
docker stats

# UI interactiva de Docker (instalado en kevin01)
lazydocker

# Reset completo de n8n (borra todos los datos)
docker stop n8n && docker rm n8n && rm -rf ./n8n_data
docker compose -f evolutionAPI.yml up -d n8n
```

---

## Mantenimiento

### Backup de la base de datos

```bash
# Exportar base de Evolution API
docker exec postgres_db pg_dump -U postgres evolution > backups/evolution_$(date +%F).sql

# Restaurar
docker exec -i postgres_db psql -U postgres evolution < backups/evolution_2025-01-01.sql
```

### Actualizar imagenes Docker

```bash
docker compose -f evolutionAPI.yml pull
docker compose -f evolutionAPI.yml up -d
```

### Rotar la API key de Evolution API

```bash
# 1. Generar nueva clave
openssl rand -hex 32

# 2. Actualizar AUTHENTICATION_API_KEY en .env

# 3. Reiniciar
docker compose -f evolutionAPI.yml restart evolution-prueba_v2
```

### Actualizar modelos de Ollama

```bash
ollama pull mistral
ollama pull qwen2.5:0.5b
```

---

## Resolucion de problemas

| Sintoma                                   | Causa probable                  | Solucion                                                              |
|-------------------------------------------|---------------------------------|-----------------------------------------------------------------------|
| `port 8080 already allocated`             | Puerto en uso                   | `docker ps` -> `docker stop <ID>` -> cambiar puerto en compose        |
| Evolution API reinicia en loop            | Credenciales DB incorrectas     | Verificar `DATABASE_CONNECTION_URI` en `.env` y reiniciar             |
| `permission denied /var/run/docker.sock`  | Usuario fuera del grupo docker  | `sudo usermod -aG docker $USER && newgrp docker`                      |
| n8n no recibe webhooks externos           | DevTunnel inactivo              | Reiniciar DevTunnel y actualizar `WEBHOOK_URL` en el contenedor       |
| Grafana no carga desde el proxy           | Apache2 detenido                | `sudo systemctl restart apache2`                                      |
| Ollama lento o sin respuesta              | Modelo muy grande para la RAM   | Usar `qwen2.5:0.5b` para tareas livianas; `mistral` para el resto    |
| Error snap al instalar ngrok              | LXC sin modulo squashfs         | Instalar binario `.tgz` directamente en `/usr/local/bin`              |

---

## Licencia

MIT (c) 2025 Kevin Alcalde — Bunker 4
