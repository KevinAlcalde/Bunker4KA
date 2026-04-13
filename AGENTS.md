# AGENTS.md

Infraestructura self-hosted del Bunker 4 / Campuseim sobre Proxmox VE. Incluye red con VLANs gestionadas por MikroTik, múltiples CTs y VMs, y un stack Docker de automatización (Evolution API, n8n, Grafana, Ollama) corriendo en CT 108 (kevin01 / 10.10.153.4).

## Comandos clave

```bash
# Conectar al bastion
ssh -p 10224 kevin@bunker4.perfeccion.ar

# Saltar a kevin01
ssh kevin@10.10.153.4

# Estado del stack Docker
docker compose -f evolutionAPI.yml ps
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# Levantar stack
docker compose -f evolutionAPI.yml up -d

# Panel Proxmox (via túnel)
ssh -N -p 10224 -L 127.0.0.1:10050:192.168.1.25:8006 kevin@bunker4.perfeccion.ar
```

## Antes de modificar

| Qué necesitás entender                        | Leer                                             |
|-----------------------------------------------|--------------------------------------------------|
| Red completa, VLANs, IPs                      | `docs/architecture/network-topology.md`          |
| Inventario de CTs y VMs en Proxmox            | `docs/architecture/proxmox-nodes.md`             |
| Todos los servicios Docker y cómo gestionarlos| `docs/services/docker-stack.md`                  |
| Evolution API (WhatsApp, webhooks, DB)        | `docs/services/evolution-api.md`                 |
| Ollama (LLM local, modelos, acceso)           | `docs/services/ollama-llm.md`                    |
| Variables de entorno del compose              | `docs/configuration/environment-variables.md`    |
| Archivos importantes del repo y kevin01       | `docs/configuration/key-files.md`                |
| Desplegar el stack desde cero                 | `docs/how-to-run/deploy.md`                      |
| Problemas comunes y soluciones                | `docs/maintenance/troubleshooting.md`            |
