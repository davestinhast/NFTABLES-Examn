# Cadenas OUTPUT y FORWARD

## Las tres cadenas principales

| Cadena | Tráfico que maneja |
|--------|-------------------|
| `input` | Tráfico entrante **hacia** el equipo |
| `output` | Tráfico saliente **desde** el equipo |
| `forward` | Tráfico que **atraviesa** el equipo (router) |

---

## Cadena OUTPUT

```bash
# Crear cadena OUTPUT
sudo nft add chain inet filter output '{ type filter hook output priority 0; policy accept; }'

# Bloquear tráfico saliente a una IP específica
sudo nft add rule inet filter output ip daddr 10.0.0.5 drop

# Permitir solo tráfico saliente HTTP/HTTPS
sudo nft add rule inet filter output tcp dport { 80, 443 } accept

# Bloquear todo lo demás saliente (política restrictiva)
sudo nft add chain inet filter output '{ type filter hook output priority 0; policy drop; }'
sudo nft add rule inet filter output ct state established,related accept
sudo nft add rule inet filter output tcp dport { 80, 443, 53 } accept
```

---

## Cadena FORWARD (para routers/gateways)

```bash
# Crear cadena FORWARD
sudo nft add chain inet filter forward '{ type filter hook forward priority 0; policy drop; }'

# Permitir forwarding de tráfico establecido
sudo nft add rule inet filter forward ct state established,related accept

# Permitir forwarding desde red interna hacia internet
sudo nft add rule inet filter forward iif eth1 oif eth0 accept

# Bloquear forwarding desde internet hacia red interna
sudo nft add rule inet filter forward iif eth0 oif eth1 ct state new drop
```

> `iif` = interface de entrada | `oif` = interface de salida
