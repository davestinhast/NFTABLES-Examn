# Contadores y Monitorización

## Contadores en reglas

```bash
# Agregar contador a una regla (cuenta paquetes y bytes)
sudo nft add rule inet filter input tcp dport 80 counter accept

# Contador con nombre (reutilizable)
sudo nft add counter inet filter http_traffic
sudo nft add rule inet filter input tcp dport 80 counter name http_traffic accept

# Ver contadores
sudo nft list counters
sudo nft list counter inet filter http_traffic
```

---

## nft monitor — tiempo real

```bash
# Ver todos los cambios de reglas en tiempo real
sudo nft monitor

# Solo cambios en reglas (no tablas/cadenas)
sudo nft monitor rules

# Solo eventos de tablas
sudo nft monitor tables

# Monitor con marcas de tiempo
sudo nft monitor | ts
```

---

## Ver estadísticas de reglas

```bash
# Listar reglas con contadores de paquetes/bytes
sudo nft list ruleset

# Output ejemplo:
# tcp dport 80 counter packets 1523 bytes 98432 accept
```

---

## Cuotas

```bash
# Bloquear después de transferir 1 GB
sudo nft add rule inet filter input ip saddr 10.0.0.5 quota over 1 gbytes drop

# Limitar a 500 MB de descarga
sudo nft add rule inet filter output quota over 500 mbytes drop
```

---

## Interfaces de red disponibles

```bash
# Ver interfaces para usar en reglas
ip link show
# o
nmcli device status

# Usar en reglas
sudo nft add rule inet filter input iif "eth0" accept
sudo nft add rule inet filter input oif "wlan0" drop
```
