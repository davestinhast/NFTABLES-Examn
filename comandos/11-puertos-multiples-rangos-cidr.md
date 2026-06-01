# Múltiples Puertos, Rangos y CIDRs

## Múltiples puertos en una sola regla

```bash
# Permitir varios puertos TCP (sintaxis de set con llaves)
sudo nft add rule inet filter input tcp dport { 22, 80, 443 } accept

# Bloquear puertos específicos
sudo nft add rule inet filter input tcp dport { 23, 135, 139, 445 } drop

# UDP múltiples puertos
sudo nft add rule inet filter input udp dport { 53, 67, 68 } accept

# Mezclar TCP y UDP con meta l4proto
sudo nft add rule inet filter input meta l4proto { tcp, udp } dport 53 accept
```

---

## Rangos de puertos

```bash
# Permitir puertos 8000 al 9000
sudo nft add rule inet filter input tcp dport 8000-9000 accept

# Bloquear puertos efímeros entrantes
sudo nft add rule inet filter input tcp dport 1024-65535 drop

# Puertos privilegiados (0-1023)
sudo nft add rule inet filter output tcp sport 0-1023 accept
```

---

## Rangos de IPs (CIDR)

```bash
# Permitir toda una subred
sudo nft add rule inet filter input ip saddr 192.168.1.0/24 accept

# Bloquear un rango completo
sudo nft add rule inet filter input ip saddr 10.0.0.0/8 drop

# Permitir solo desde red de gestión
sudo nft add rule inet filter input ip saddr 172.16.0.0/12 tcp dport 22 accept

# Bloquear clase C específica
sudo nft add rule inet filter input ip saddr 203.0.113.0/24 drop
```

---

## Combinaciones IP + Puerto

```bash
# Solo esta IP puede acceder al puerto 3306 (MySQL)
sudo nft add rule inet filter input ip saddr 192.168.1.50 tcp dport 3306 accept

# Bloquear esa subred en puerto 22 pero permitir el resto
sudo nft add rule inet filter input ip saddr 10.10.0.0/16 tcp dport 22 drop
sudo nft add rule inet filter input tcp dport 22 accept

# Permitir HTTP solo desde LAN
sudo nft add rule inet filter input ip saddr 192.168.0.0/16 tcp dport 80 accept
sudo nft add rule inet filter input tcp dport 80 drop
```

---

## Sets con CIDRs (interval sets)

```bash
# Crear set con soporte para rangos/CIDRs
sudo nft add set inet filter trusted_nets { type ipv4_addr\; flags interval\; }

# Agregar CIDRs al set
sudo nft add element inet filter trusted_nets { 192.168.0.0/24, 10.0.0.0/8 }

# Usar el set en una regla
sudo nft add rule inet filter input ip saddr @trusted_nets accept
```
