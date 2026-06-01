# NAT — DNAT, SNAT y Masquerade

## Estructura de NAT en nftables

NAT requiere su propia tabla (`ip nat`) con cadenas especiales de tipo `nat`.

```bash
# Crear tabla NAT
sudo nft add table ip nat

# Crear cadena PREROUTING (para DNAT — antes de enrutamiento)
sudo nft add chain ip nat prerouting '{ type nat hook prerouting priority -100; }'

# Crear cadena POSTROUTING (para SNAT/Masquerade — después de enrutamiento)
sudo nft add chain ip nat postrouting '{ type nat hook postrouting priority 100; }'
```

---

## DNAT — Destination NAT (Port Forwarding)

Redirige tráfico entrante a otra IP/puerto (útil para exponer servicios internos).

```bash
# Redirigir puerto 80 externo al servidor interno 192.168.1.10:80
sudo nft add rule ip nat prerouting tcp dport 80 dnat to 192.168.1.10

# Redirigir con cambio de puerto (80 externo → 8080 interno)
sudo nft add rule ip nat prerouting tcp dport 80 dnat to 192.168.1.10:8080

# DNAT solo para tráfico que entra por eth0
sudo nft add rule ip nat prerouting iif eth0 tcp dport 443 dnat to 192.168.1.20:443
```

---

## SNAT — Source NAT

Cambia la IP de origen de los paquetes salientes (IP estática conocida).

```bash
# SNAT: cambiar IP origen a 203.0.113.5 para tráfico saliente por eth0
sudo nft add rule ip nat postrouting oif eth0 snat to 203.0.113.5

# SNAT para red interna específica
sudo nft add rule ip nat postrouting ip saddr 192.168.1.0/24 oif eth0 snat to 203.0.113.5
```

---

## Masquerade (SNAT dinámico)

Igual que SNAT pero usa automáticamente la IP de la interfaz de salida (ideal para IPs dinámicas / DHCP).

```bash
# Masquerade para toda la red interna
sudo nft add rule ip nat postrouting oif eth0 masquerade

# Masquerade solo para subred específica
sudo nft add rule ip nat postrouting ip saddr 192.168.0.0/24 oif eth0 masquerade
```

> **SNAT vs Masquerade:**
> - `snat to X.X.X.X` → IP fija conocida (servidores con IP estática)
> - `masquerade` → IP dinámica (conexiones domésticas, DHCP)

---

## Habilitar IP Forwarding (necesario para NAT)

```bash
# Temporal
sudo sysctl -w net.ipv4.ip_forward=1

# Permanente
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```
