# Nodos Proxmox

Inventario de todos los contenedores (CT) y máquinas virtuales (VM) en el Proxmox VE del Bunker 4.

---

## Host físico

| Campo     | Valor                                 |
|-----------|---------------------------------------|
| CPU       | AMD Ryzen 7 5700G (16 cores / 32 threads) |
| Storage   | ZFS pool `Tanque03`                   |
| Red       | Modem Movistar → MikroTik → Proxmox (eno1 → vmbr0 / 192.168.1.25) |
| Panel     | https://bunker4.perfeccion.ar:10050   |

---

## Contenedores LXC

| CT  | Nombre              | OS            | IP           | VLAN | RAM    | Storage | Función                          |
|-----|---------------------|---------------|--------------|------|--------|---------|----------------------------------|
| 100 | testvlan3           | —             | 10.10.153.2  | 153  | —      | —       | Playground padawans — revisar DNS con Lautaro |
| 102 | nginx-proxy-manager | ArchLinux     | 10.10.150.7  | 150  | —      | —       | Proxy reverso HTTPS `*.perfeccion.ar` |
| 103 | bastion-arch        | Arch Linux    | 10.10.150.8  | 150  | 512 MB | 7.8 GB LVM | Gateway SSH — único punto de entrada externo |
| 106 | testvlan            | Ubuntu 22.04  | 192.168.89.52| lab  | —      | —       | Servidores de escuela (Rails, Streamlit, etc.) |
| 107 | testvlan2           | Ubuntu 22.04  | 10.10.151.2  | 151  | —      | —       | Laboratorio de contenedores      |
| 108 | kevin01             | Ubuntu 24.04  | 10.10.153.4  | 153  | 2 GB   | 20 GB ZFS | Stack Docker personal (ver `docs/services/`) |

---

## Máquinas Virtuales

| VM  | Nombre        | OS           | IP              | VLAN | Función                                      |
|-----|---------------|--------------|-----------------|------|----------------------------------------------|
| 105 | lubuntu-tools | Ubuntu       | 10.10.151.3     | 151  | Tools, browsers, Winbox, servidores menores  |
| 200 | k8s-ctrl      | Ubuntu 24.04 | 192.168.89.196  | 152* | Control plane del cluster Kubernetes         |
| 201 | k8s-node      | Ubuntu 24.04 | —               | 152  | Worker node K8s — Guest not running, pendiente migración |
| 300 | Ainsophic     | Alma 9.5     | 10.10.151.4     | 151  | Raro: aparece en red por 192.168.89.22       |

> `*` VM 200 aún figura en 192.168.89.196 — pendiente migración completa a VLAN 152.

---

## Acceso al panel Proxmox

```bash
# Web UI (desde la red interna o via túnel SSH)
https://bunker4.perfeccion.ar:10050
```

Para acceder via túnel SSH desde máquina local:

```bash
ssh -N -p 10224 \
    -L 127.0.0.1:10050:192.168.1.25:8006 \
    kevin@bunker4.perfeccion.ar
# Luego abrir: https://localhost:10050
```
