# Stack Docker — kevin01

Describe todos los servicios que corren en CT 108 (kevin01 / 10.10.153.4), la VM principal de automatización.

---

## Resumen del stack

| Servicio        | Imagen                                    | Puerto host | Estado   | Función                        |
|-----------------|-------------------------------------------|-------------|----------|--------------------------------|
| Evolution API   | `atendai/evolution-api:v2.1.1-homolog`   | 8080        | Activo   | API WhatsApp (ver evolution-api.md) |
| n8n             | `n8nio/n8n:latest`                        | 5678        | Activo   | Automatización low-code        |
| Grafana         | `grafana/grafana:latest`                  | 3000        | Activo   | Dashboards y monitoreo         |
| Portainer CE    | `portainer/portainer-ce`                  | 9000        | Activo   | UI de gestión Docker           |
| PostgreSQL      | `postgres:14.13`                          | 5432        | Activo   | Base de datos principal        |
| pgAdmin 4       | `dpage/pgadmin4:latest`                   | 5433→80     | Activo   | UI web para PostgreSQL         |
| Redis           | `redis:latest`                            | interno     | Activo   | Cache y cola de Evolution API  |
| Ollama          | servicio systemd                          | 11434 (localhost) | Activo | Inferencia LLM local      |
| Apache2         | servicio systemd                          | 80          | Activo   | Proxy reverso hacia Grafana    |
| Node-RED        | `nodered/node-red:latest`                 | 1880        | Detenido | Flujos IoT (pausado)           |

---

## Red Docker

Todos los contenedores usan la red externa `red_gral`:

```bash
# Crear la red (solo primera vez)
docker network create red_gral
```

En el compose file se referencia como:
```yaml
networks:
  rede_geral:
    external: true
    name: red_gral
```

---

## Volúmenes persistentes

| Volumen               | Servicio       | Descripción                        |
|-----------------------|----------------|------------------------------------|
| `redis_data`          | Redis          | Datos AOF persistidos              |
| `postgres_data`       | PostgreSQL     | Base de datos completa             |
| `evolution_instancesv2` | Evolution API | Instancias y sesiones WhatsApp   |

---

## n8n

- Puerto: `5678`
- Webhook URL activa: `https://kp3pxp3p-5678.usw3.devtunnels.ms/` (Microsoft DevTunnels)
- Si el DevTunnel cae, actualizar `WEBHOOK_URL` y reiniciar el contenedor

```bash
# Reiniciar n8n
docker restart n8n

# Reset completo (borra todos los datos)
docker stop n8n && docker rm n8n && rm -rf ./n8n_data
docker compose -f evolutionAPI.yml up -d n8n
```

---

## Grafana

- Puerto interno: `3000`
- URL pública: `http://grafana.kevin.perfeccion.ar` (via Apache2 proxy)
- Usuario admin: `grafkev`

Apache2 config en `/etc/apache2/sites-enabled/`:
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

## Portainer

- Puerto: `9000`
- Gestión visual de todos los contenedores Docker en kevin01

---

## Comandos útiles

```bash
# Estado de todos los contenedores
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# Logs en tiempo real
docker logs -f evolution_v2
docker logs -f n8n

# Shell de un contenedor
docker exec -it evolution_v2 bash
docker exec -it n8n sh

# Reiniciar un servicio
docker restart grafana
docker restart n8n

# Recursos en tiempo real
docker stats

# UI interactiva (instalado en kevin01)
lazydocker

# Actualizar imágenes
docker compose -f evolutionAPI.yml pull
docker compose -f evolutionAPI.yml up -d
```
