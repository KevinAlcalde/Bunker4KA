# Archivos Clave del Repositorio

Archivos importantes que un agente o administrador debe conocer antes de modificar el stack.

---

## Raíz del repositorio

| Archivo / Carpeta       | Descripción                                                       |
|-------------------------|-------------------------------------------------------------------|
| `evolutionAPI.yml`      | Docker Compose principal — define todos los servicios del stack   |
| `README.md`             | Documentación general del proyecto                                |
| `.sdd.json`             | Config de SDD: paths de docs, specs, scripts de test             |
| `docs/`                 | Base de conocimiento para agentes (este directorio)               |
| `Group 9251.png`        | Diagrama de arquitectura de red (imagen)                          |

---

## En kevin01 (10.10.153.4)

| Path                                          | Descripción                                              |
|-----------------------------------------------|----------------------------------------------------------|
| `/home/kevin/automation/docker-compose.yml`   | Compose de servicios adicionales (n8n, Grafana, etc.)    |
| `/home/kevin/automation/evolutionAPI.yml`     | Compose de Evolution API (copia desplegada)              |
| `/etc/apache2/sites-enabled/`                 | Config Apache2 proxy para Grafana                        |
| `/etc/systemd/system/ollama.service`          | Servicio systemd de Ollama                               |

---

## Compose files

El repositorio tiene **un solo compose file** activo: `evolutionAPI.yml`.

Incluye:
- `evolution_v2` — Evolution API v2.1.1
- `postgres` — PostgreSQL 14.13
- `pgadmin` — UI web de PostgreSQL
- `redis` — Cache/cola

Servicios adicionales (n8n, Grafana, Portainer, Node-RED) se gestionan con un compose separado en kevin01, no versionado en este repo actualmente.

---

## Scripts (pendientes de agregar)

| Script                        | Descripción                                            |
|-------------------------------|--------------------------------------------------------|
| `scripts/run-tests.sh`        | Referenciado en `.sdd.json` — aún no existe           |
| `scripts/clean-test-cache.sh` | Referenciado en `.sdd.json` — aún no existe           |
