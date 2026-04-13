# Ollama — LLM Local

Ollama corre como servicio systemd en kevin01, escuchando solo en `127.0.0.1:11434`. No está expuesto externamente.

---

## Modelos instalados

| Modelo           | Tamaño | Uso recomendado                     |
|------------------|--------|-------------------------------------|
| `mistral:latest` | 4.4 GB | Chat, razonamiento general          |
| `qwen2.5:0.5b`   | 397 MB | Tareas livianas, pruebas rápidas    |

---

## Gestión del servicio

```bash
# Estado del servicio
sudo systemctl status ollama

# Reiniciar
sudo systemctl restart ollama

# Logs
journalctl -u ollama -f
```

---

## Comandos Ollama

```bash
# Listar modelos
ollama list

# Chat interactivo
ollama run mistral
ollama run qwen2.5:0.5b

# Agregar modelo
ollama pull llama3.2

# Llamada REST (desde kevin01)
curl http://localhost:11434/api/generate \
  -d '{"model":"mistral","prompt":"Hola","stream":false}'
```

---

## Acceso desde máquina local

Ollama no tiene puerto expuesto. Usar túnel SSH:

```bash
ssh -N -p 10224 \
    -L 127.0.0.1:11434:10.10.153.4:11434 \
    kevin@bunker4.perfeccion.ar
# Luego: curl http://localhost:11434/api/generate ...
```

---

## Consideraciones de recursos

- `mistral:latest` requiere ~5 GB RAM — usar solo cuando el nodo tenga headroom
- `qwen2.5:0.5b` es adecuado para embeddings rápidos o pruebas sin carga
- Si Ollama no responde, verificar que el servicio systemd esté activo y que haya RAM disponible
