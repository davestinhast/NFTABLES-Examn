# Reglas de Firewall

## Tráfico local

```bash
# Permitir loopback (localhost)
sudo nft add rule inet filter input iif lo accept
```
> Las aplicaciones locales funcionarán correctamente.

---

## SSH (puerto 22)

```bash
# Permitir SSH
sudo nft add rule inet filter input tcp dport 22 accept

# Permitir SSH con límite de tasa (anti-bruteforce)
sudo nft add rule inet filter input tcp dport 22 limit rate 5/minute accept
```

---

## HTTP / HTTPS

```bash
# Permitir HTTP (puerto 80)
sudo nft add rule inet filter input tcp dport 80 accept

# Permitir HTTPS (puerto 443)
sudo nft add rule inet filter input tcp dport 443 accept
```

---

## Bloquear IP específica

```bash
sudo nft add rule inet filter input ip saddr 192.168.1.100 drop
```
> La IP `192.168.1.100` no podrá conectarse al servidor.

---

## ICMP / Ping

```bash
# Permitir ping
sudo nft add rule inet filter input icmp type echo-request accept

# Bloquear ping (hacer servidor invisible)
sudo nft add rule inet filter input icmp type echo-request drop
```

---

## NAT

```bash
# Crear tabla NAT
sudo nft add table ip nat
```
