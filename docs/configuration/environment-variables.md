# Variables de Entorno

Variables configuradas en `evolutionAPI.yml`. Las credenciales van en el compose directamente (no hay `.env` file actualmente).

---

## PostgreSQL

| Variable            | Valor por defecto         | Descripción                  |
|---------------------|---------------------------|------------------------------|
| `POSTGRES_USER`     | `postgres`                | Usuario de la DB             |
| `POSTGRES_PASSWORD` | *(reemplazar)*            | Contraseña del superusuario  |

---

## pgAdmin

| Variable                   | Descripción                  |
|----------------------------|------------------------------|
| `PGADMIN_DEFAULT_EMAIL`    | Email para login en pgAdmin  |
| `PGADMIN_DEFAULT_PASSWORD` | Contraseña de pgAdmin        |

---

## Evolution API

### Servidor y auth
| Variable                              | Valor actual           | Descripción                        |
|---------------------------------------|------------------------|------------------------------------|
| `SERVER_URL`                          | `http://localhost:8080`| URL pública de la API              |
| `AUTHENTICATION_TYPE`                 | `apikey`               | Método de autenticación            |
| `AUTHENTICATION_API_KEY`              | *(reemplazar)*         | Clave API — generar con `openssl rand -hex 32` |
| `AUTHENTICATION_EXPOSE_IN_FETCH_INSTANCES` | `true`           | Expone la key al listar instancias |

### General
| Variable                      | Valor         | Descripción                            |
|-------------------------------|---------------|----------------------------------------|
| `LANGUAGE`                    | `pt-BR`       | Idioma de Evolution (mensajes sistema) |
| `CONFIG_SESSION_PHONE_CLIENT` | `InovaBoost`  | Nombre del cliente WhatsApp            |
| `CONFIG_SESSION_PHONE_NAME`   | `chrome`      | Navegador simulado                     |

### Base de datos
| Variable                        | Descripción                                      |
|---------------------------------|--------------------------------------------------|
| `DATABASE_ENABLED`              | `true`                                           |
| `DATABASE_PROVIDER`             | `postgresql`                                     |
| `DATABASE_CONNECTION_URI`       | `postgresql://postgres:<PWD>@postgres_db:5432/evolution` |
| `DATABASE_SAVE_DATA_INSTANCE`   | `true` — persiste configuración de instancias   |
| `DATABASE_SAVE_DATA_NEW_MESSAGE`| `true` — guarda mensajes entrantes              |
| `DATABASE_SAVE_MESSAGE_UPDATE`  | `true` — guarda actualizaciones de mensajes     |
| `DATABASE_SAVE_DATA_CONTACTS`   | `true`                                           |
| `DATABASE_SAVE_DATA_CHATS`      | `true`                                           |

### Redis
| Variable                        | Valor                                | Descripción                  |
|---------------------------------|--------------------------------------|------------------------------|
| `CACHE_REDIS_ENABLED`           | `true`                               | Habilita cache Redis         |
| `CACHE_REDIS_URI`               | `redis://redis_service:6379/2`       | DB 2 de Redis                |
| `CACHE_REDIS_PREFIX_KEY`        | `evolution`                          | Prefijo de keys              |
| `CACHE_REDIS_SAVE_INSTANCES`    | `false`                              |                              |
| `CACHE_LOCAL_ENABLED`           | `false`                              | Cache local desactivado      |

### Instancias
| Variable            | Valor | Descripción                                    |
|---------------------|-------|------------------------------------------------|
| `DEL_INSTANCE`      | `2`   | Tiempo (min) para eliminar instancias inactivas |
| `DEL_TEMP_INSTANCES`| `false` | No eliminar instancias temporales             |

---

## Generar credenciales

```bash
# API key segura
openssl rand -hex 32

# Password aleatoria
openssl rand -base64 24
```
