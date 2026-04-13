# Resolución de Problemas

Síntomas comunes, causas y soluciones para el stack del Bunker 4.

---

## Docker / Servicios

| Síntoma                                    | Causa probable                   | Solución                                                                 |
|--------------------------------------------|----------------------------------|--------------------------------------------------------------------------|
| `port 8080 already allocated`              | Puerto en uso                    | `docker ps` → `docker stop <ID>` → cambiar puerto en compose            |
| Evolution API reinicia en loop             | Credenciales DB incorrectas      | Verificar `DATABASE_CONNECTION_URI` en el compose y reiniciar           |
| `permission denied /var/run/docker.sock`   | Usuario fuera del grupo docker   | `sudo usermod -aG docker $USER && newgrp docker`                        |
| n8n no recibe webhooks externos            | DevTunnel inactivo               | Reiniciar DevTunnel y actualizar `WEBHOOK_URL` en el contenedor         |
| Grafana no carga desde el proxy            | Apache2 detenido                 | `sudo systemctl restart apache2`                                        |
| Ollama lento o sin respuesta               | Modelo muy grande para la RAM    | Usar `qwen2.5:0.5b` para tareas livianas; `mistral` para el resto       |
| Error snap al instalar ngrok               | LXC sin módulo squashfs          | Instalar binario `.tgz` directamente en `/usr/local/bin`                |

---

## Red / Acceso SSH

| Síntoma                              | Causa probable                        | Solución                                          |
|--------------------------------------|---------------------------------------|---------------------------------------------------|
| SSH al bastion no conecta            | Puerto 10224 bloqueado o MikroTik caído | Verificar reglas NAT en MikroTik                 |
| No hay conectividad en VLAN 151      | MikroTik no enruta hacia esa VLAN    | Revisar interfaces y bridge en MikroTik           |
| VM 300 (Ainsophic) aparece en 192.168.89.22 | Error de VLAN tag            | Verificar configuración de VLAN en Proxmox para VM 300 |
| CT 100 (testvlan3) no resuelve DNS   | DNS mal configurado                   | Revisar con Lautaro — posible problema en resolv.conf |

---

## Proxmox

| Síntoma                          | Solución                                              |
|----------------------------------|-------------------------------------------------------|
| Panel web inaccesible            | Acceder via túnel SSH: `-L 10050:192.168.1.25:8006`  |
| CT no arranca                    | Verificar recursos disponibles en el host (RAM, ZFS) |
| ZFS pool degradado               | `zpool status Tanque03` → identificar disco fallido  |

---

## Backups PostgreSQL

```bash
# Backup Evolution API
docker exec postgres_db pg_dump -U postgres evolution > backups/evolution_$(date +%F).sql

# Restore
docker exec -i postgres_db psql -U postgres evolution < backups/evolution_YYYY-MM-DD.sql
```
