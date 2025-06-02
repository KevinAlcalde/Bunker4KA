# 🛡️ Bunker 4 – Plataforma EvolutionAPI + N8N + Portainer

Implementación auto-contenida en **Proxmox** (LXC) que expone —de forma segura— los siguientes
servicios Dockerizados:

| Servicio     | Imagen                         | Puerto interno | Puerto host¹ | Función                      |
|--------------|-------------------------------|---------------:|-------------:|------------------------------|
| EvolutionAPI | `atendai/evolution-api:2.1`   | 8080           | *(túnel)*    | API WhatsApp inteligente     |
| N8N          | `n8nio/n8n:latest`            | 5678           | 5678         | Low-code automation          |
| Redis        | `redis:latest`                | 6379           | —            | Broker de colas / cache      |
| PostgreSQL   | `postgres:14`                 | 5432           | —            | Persistencia EvolutionAPI    |
| pgAdmin 4    | `dpage/pgadmin4:latest`       | 80/443         | 5433 (TLS)   | GUI DB                       |
| Portainer CE | `portainer/portainer-ce`      | 9000/9443      | 9000/9443    | UI gestión contenedores      |

> ¹ **Nunca** exponemos puertos públicos; todo viaja por *SSH local-port-forward*.

---

## 📑 Tabla de contenidos
1. [Arquitectura](#arquitectura)
2. [Prerrequisitos](#prerrequisitos)
3. [Despliegue rápido](#despliegue-rápido)
4. [Túneles y acceso seguro](#túneles-y-acceso-seguro)
5. [Variables de entorno](#variables-de-entorno)
6. [Comandos útiles](#comandos-útiles)
7. [Solución de problemas](#solución-de-problemas)
8. [Mantenimiento](#mantenimiento)
9. [Licencia](#licencia)

---

## Arquitectura

![Tipología Bunker 4](docs/images/bunker4_topologia.png)

| # | Componente | Descripción |
|---|------------|-------------|
| 1 | **Hetzner 1 / Bastion** | Servidor Audialux expuesto a Internet (`ssh :10224`). |
| 2 | **VM kevin@perfeccion.ar** | Nodo intermediario donde se crean los túneles. |
| 3 | **Proxmox Host** | Contiene LXC `kvn01` (`10.10.153.4`). |
| 4 | **Docker Stack** | Servicios listados arriba sobre una red interna. |

---

## Prerequisitos

| Recurso | Versión mínima |
|---------|----------------|
| Ubuntu 22.04 LTS (VM/LXC) | — |
| Docker Engine | `24.x` |
| Docker Compose Plugin | `v2.27+` |
| Acceso SSH bastion | Puerto `10224` |
| DNS → Nginx-Proxy-Manager (opcional) | `*.perfeccion.ar` |

---

## Despliegue rápido

# 1 – Clona este repo dentro del contenedor LXC
git clone https://github.com/<ORG>/<REPO>.git
cd <REPO>

# 2 – Ajusta variables
cp .env.example .env && nano .env

# 3 – Arranca el stack
docker compose pull
docker compose up -d

# 4 – Verifica
docker compose ps
Persistencia

evolution_store y evolution_instances → datos WhatsApp

postgres_data → base EvolutionAPI
---
Túneles y acceso seguro
Crear túnel SSH (VM → Bastion → Contenedor)

# Ejemplo: forward local 50000 → contenedor 22
ssh -N -p 10224 \
    -L 127.0.0.1:50000:10.10.153.4:22 \
    kevin@bunker4.perfeccion.ar
Servicio	 Puerto local	 Destino interno
SSH LXC	        50000	       10.10.153.4:22
N8N	            50001	       10.10.153.4:5678
EvolutionAPI	50002	       10.10.153.4:8080
Portainer	    50003	       10.10.153.4:9000

Cliente local → ssh -p 50000 kevin@localhost o http://localhost:50002.

# Alternativas 
Método	      Ventaja	                           Nota
Cloudflared	  DNS *.trycloudflare.com	        cloudflared tunnel create + YAML
ngrok free	Fácil, 1 túnel simultáneo	 subdominio cambia en cada restart
---
# Variables de entorno
AUTHENTICATION_API_KEY=<clave_hex_64>
TZ=America/Argentina/Buenos_Aires

POSTGRES_USER=postgres
POSTGRES_PASSWORD=<pass_db>
POSTGRES_DB=evolution

PGADMIN_DEFAULT_EMAIL=admin@local
PGADMIN_DEFAULT_PASSWORD=<pass_pgadmin>
Generar clave: openssl rand -hex 32
---
# Comandos útiles
Acción	             Comando
Listar contenedores	     docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
Logs EvolutionAPI	     docker logs -f evolution_api
Entrar al contenedor	 docker exec -it evolution_api sh
Reiniciar Portainer  	 docker restart portainer
Reset credenciales N8N	 docker stop n8n && docker rm n8n && rm -rf ./n8n_data && docker compose up -d n8n
Métricas live	         docker stats / htop
---
# Solución de problemas
Síntoma	                                        Causa	                                        Fix
port 8080 already allocated	              8080 ya en uso	docker ps        docker stop <ID>, cambiar puerto compose
EvolAPI reinicia (P1000)	              Credenciales DB incorrectas	     Revisar .env y reiniciar EvolutionAPI
permission denied /var/run/docker.sock 	  Usuario fuera del grupo docker	 sudo usermod -aG docker $USER && newgrp docker
Error snap (ngrok)	LXC sin squashfs	  Instalar binario .tgz en /usr/local/bin
---
# Mantenimiento
## Backup DB
docker exec postgres_db pg_dump -U postgres evolution > backups/evolution_$(date +%F).sql

## Actualizar imágenes
docker compose pull && docker compose up -d

## Rotar API key
openssl rand -hex 32  # editar .env
docker compose restart evolution_api
## Túneles y acceso seguro
Crear túnel SSH (VM → Bastion → Contenedor)

# Ejemplo: forward local 50000 → contenedor 22
ssh -N -p 10224 \
    -L 127.0.0.1:50000:10.10.153.4:22 \
    kevin@bunker4.perfeccion.ar
Servicio	Puerto local	Destino interno
SSH LXC	50000	10.10.153.4:22
N8N	50001	10.10.153.4:5678
EvolutionAPI	50002	10.10.153.4:8080
Portainer	50003	10.10.153.4:9000

Cliente local → ssh -p 50000 kevin@localhost o http://localhost:50002.
---
# Alternativas
Método	Ventaja	Nota
Cloudflared	DNS *.trycloudflare.com	cloudflared tunnel create + YAML
ngrok free	Fácil, 1 túnel simultáneo	subdominio cambia en cada restart
---
Variables de entorno
env
Copy
Edit
AUTHENTICATION_API_KEY=<clave_hex_64>
TZ=America/Argentina/Buenos_Aires

POSTGRES_USER=postgres
POSTGRES_PASSWORD=<pass_db>
POSTGRES_DB=evolution

PGADMIN_DEFAULT_EMAIL=admin@local
PGADMIN_DEFAULT_PASSWORD=<pass_pgadmin>
Generar clave: openssl rand -hex 32
---
# Comandos útiles
Acción                   	Comando
Listar contenedores 	docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
Logs EvolutionAPI	    docker logs -f evolution_api
Entrar al contenedor	docker exec -it evolution_api sh
Reiniciar Portainer	    docker restart portainer
Reset credenciales N8N	docker stop n8n && docker rm n8n && rm -rf ./n8n_data && docker compose up -d n8n
Métricas live	        docker stats / htop
---
Solución de problemas
Síntoma	Causa	Fix
port 8080 already allocated	8080 ya en uso	docker ps, docker stop <ID>, cambiar puerto compose
EvolAPI reinicia (P1000)	Credenciales DB incorrectas	Revisar .env y reiniciar EvolutionAPI
permission denied /var/run/docker.sock	Usuario fuera del grupo docker	sudo usermod -aG docker $USER && newgrp docker
Error snap (ngrok)	LXC sin squashfs	Instalar binario .tgz en /usr/local/bin
---
# Mantenimiento
## Backup DB
docker exec postgres_db pg_dump -U postgres evolution > backups/evolution_$(date +%F).sql
---
## Actualizar imágenes
docker compose pull && docker compose up -d
---
## Rotar API key
openssl rand -hex 32  # editar .env
docker compose restart evolution_api
## Licencia
MIT • © 2025 Bunker 4
