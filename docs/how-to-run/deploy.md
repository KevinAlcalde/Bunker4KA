# Despliegue del Stack

Pasos para levantar el stack Docker en kevin01 desde cero.

---

## Requisitos previos

| Componente           | Versión mínima |
|----------------------|----------------|
| Ubuntu 22.04+ LXC/VM | —              |
| Docker Engine        | 24.x           |
| Docker Compose Plugin| v2.27+         |
| Acceso SSH al bastion| Puerto 10224   |

---

## 1. Conectarse a kevin01

```bash
# Opción A — via bastion (dos saltos)
ssh -p 10224 kevin@bunker4.perfeccion.ar
ssh kevin@10.10.153.4

# Opción B — ProxyJump directo
ssh -J kevin@bunker4.perfeccion.ar:10224 kevin@10.10.153.4
```

---

## 2. Clonar el repositorio

```bash
git clone https://github.com/KevinAlcalde/Bunker4KA.git
cd Bunker4KA
```

---

## 3. Crear la red Docker externa

Solo necesario la primera vez:

```bash
docker network create red_gral
```

---

## 4. Configurar credenciales

Editar directamente en `evolutionAPI.yml` y reemplazar los placeholders:

| Placeholder                    | Reemplazar con                         |
|--------------------------------|----------------------------------------|
| `(CONTRASEÑA_POSTGRES_AQUI)`   | Password segura para PostgreSQL        |
| `(CONTRASEÑA_PGADMIN)`         | Password para pgAdmin                  |
| `(TU APIKEY)`                  | `openssl rand -hex 32`                 |
| `admin@gmail.com`              | Email real para pgAdmin                |

---

## 5. Levantar el stack

```bash
docker compose -f evolutionAPI.yml up -d
```

---

## 6. Verificar estado

```bash
docker compose -f evolutionAPI.yml ps
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

---

## Acceso a servicios (desde red interna o túnel SSH)

| Servicio      | URL / Dirección                          |
|---------------|------------------------------------------|
| Evolution API | `http://10.10.153.4:8080`               |
| pgAdmin       | `http://10.10.153.4:5433`               |
| n8n           | `http://10.10.153.4:5678`               |
| Grafana       | `http://grafana.kevin.perfeccion.ar`    |
| Portainer     | `http://10.10.153.4:9000`               |

---

## Túneles SSH para acceso local

```bash
ssh -N -p 10224 \
    -L 127.0.0.1:8080:10.10.153.4:8080 \
    -L 127.0.0.1:5678:10.10.153.4:5678 \
    -L 127.0.0.1:9000:10.10.153.4:9000 \
    -L 127.0.0.1:5433:10.10.153.4:5433 \
    kevin@bunker4.perfeccion.ar
```

| Servicio      | Puerto local | URL local                  |
|---------------|-------------|----------------------------|
| Evolution API | 8080        | http://localhost:8080       |
| n8n           | 5678        | http://localhost:5678       |
| Portainer     | 9000        | http://localhost:9000       |
| pgAdmin       | 5433        | http://localhost:5433       |
| PostgreSQL    | 5432        | psql en localhost:5432      |
| Ollama API    | 11434       | http://localhost:11434      |

---

## Config VSCode Remote SSH

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
