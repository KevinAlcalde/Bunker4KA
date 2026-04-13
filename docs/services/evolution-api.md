# Evolution API

Servicio de mensajería WhatsApp que corre en kevin01. Versión activa: `v2.1.1-homolog`.

---

## Config base

| Campo              | Valor                                       |
|--------------------|---------------------------------------------|
| Imagen             | `atendai/evolution-api:v2.1.1-homolog`      |
| Container name     | `evolution_v2`                              |
| Puerto             | `8080`                                      |
| Auth type          | `apikey`                                    |
| Idioma             | `pt-BR` (configurado en compose)            |
| Cliente WhatsApp   | `InovaBoost` / Chrome                       |
| DB                 | PostgreSQL (`evolution` database)           |
| Cache              | Redis (`redis://redis_service:6379/2`)      |

---

## Integraciones habilitadas

| Módulo    | Estado      | Notas                                   |
|-----------|-------------|-----------------------------------------|
| Typebot   | Habilitado  | `TYPEBOT_API_VERSION=latest`            |
| Webhooks  | Habilitado  | Mayoría de eventos activados (ver vars) |
| RabbitMQ  | Desactivado | Config presente pero `ENABLED=false`    |
| Chatwoot  | Desactivado | Config presente pero `ENABLED=false`    |
| OpenAI    | Desactivado | `OPENAI_ENABLED=false`                  |
| Dify      | Desactivado | `DIFY_ENABLED=false`                    |
| S3/MinIO  | Desactivado | `S3_ENABLED=false`                      |

---

## Dependencias

```yaml
depends_on:
  - redis
  - postgres
```

La API no arranca si Redis o PostgreSQL no están corriendo.

---

## Webhooks activos por defecto

Los siguientes eventos disparan webhook hacia la URL configurada:

- `QRCODE_UPDATED`, `MESSAGES_SET`, `MESSAGES_UPSERT`, `MESSAGES_EDITED`
- `MESSAGES_UPDATE`, `MESSAGES_DELETE`, `SEND_MESSAGE`
- `CONTACTS_SET/UPSERT/UPDATE`
- `CHATS_SET/UPSERT/UPDATE/DELETE`
- `GROUPS_UPSERT/UPDATE`, `GROUP_PARTICIPANTS_UPDATE`
- `CONNECTION_UPDATE`, `CALL`, `LABELS_EDIT/ASSOCIATION`
- `PRESENCE_UPDATE`

> Webhook global desactivado (`WEBHOOK_GLOBAL_ENABLED=false`). Los webhooks se configuran por instancia.

---

## Comandos frecuentes

```bash
# Ver logs en tiempo real
docker logs -f evolution_v2

# Reiniciar
docker restart evolution_v2

# Rotar API key
openssl rand -hex 32
# → actualizar AUTHENTICATION_API_KEY en el compose
docker compose -f evolutionAPI.yml restart evolution_v2

# Shell interactivo
docker exec -it evolution_v2 bash
```

---

## Base de datos

- DB name: `evolution`
- URI: `postgresql://postgres:<PASSWORD>@postgres_db:5432/evolution`
- Guarda: instancias, mensajes nuevos, actualizaciones, contactos, chats

```bash
# Backup
docker exec postgres_db pg_dump -U postgres evolution > backups/evolution_$(date +%F).sql

# Restore
docker exec -i postgres_db psql -U postgres evolution < backups/evolution_YYYY-MM-DD.sql
```
