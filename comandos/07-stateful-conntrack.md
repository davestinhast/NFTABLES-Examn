# Stateful Firewall — Conntrack (ct state)

> Este es uno de los conceptos más importantes. Un firewall **stateful** rastrea el estado de las conexiones para permitir tráfico de retorno automáticamente.

## Estados de conexión

| Estado | Significado |
|--------|-------------|
| `new` | Primera conexión (SYN) |
| `established` | Conexión ya activa con respuestas |
| `related` | Conexión relacionada (ej: FTP data) |
| `invalid` | Paquete que no pertenece a ninguna conexión |

## Reglas conntrack esenciales

```bash
# Permitir tráfico de conexiones ya establecidas y relacionadas (retorno)
sudo nft add rule inet filter input ct state established,related accept

# Bloquear paquetes inválidos
sudo nft add rule inet filter input ct state invalid drop

# Permitir solo conexiones nuevas + establecidas/relacionadas en SSH
sudo nft add rule inet filter input tcp dport 22 ct state new,established accept
```

## Configuración completa con stateful

```bash
sudo nft flush ruleset
sudo nft add table inet filter

# Cadena INPUT con política DROP
sudo nft add chain inet filter input '{ type filter hook input priority 0; policy drop; }'

# Loopback
sudo nft add rule inet filter input iif lo accept

# Conntrack: permitir retorno de conexiones
sudo nft add rule inet filter input ct state established,related accept

# Conntrack: bloquear inválidos
sudo nft add rule inet filter input ct state invalid drop

# Servicios nuevos
sudo nft add rule inet filter input tcp dport 22 ct state new accept
sudo nft add rule inet filter input tcp dport 80 ct state new accept
sudo nft add rule inet filter input tcp dport 443 ct state new accept
```

> ⚠️ Sin `ct state established,related accept`, las respuestas del servidor también serían bloqueadas por la política DROP.
