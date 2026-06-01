# Ruleset Completo — Ejemplo de Servidor

> Este es un ejemplo de configuración completa y funcional para un servidor Linux con nftables. Cubre todos los conceptos del examen en un solo archivo.

## Configuración como archivo /etc/nftables.conf

```bash
#!/usr/sbin/nft -f

# Limpiar todo antes de aplicar
flush ruleset

# ============================================================
# TABLA PRINCIPAL DE FILTRADO (IPv4 + IPv6)
# ============================================================
table inet filter {

    # --- Blacklist de IPs bloqueadas ---
    set blacklist {
        type ipv4_addr
        flags interval
        elements = { 10.0.0.5, 192.168.50.0/24 }
    }

    # --- Cadena INPUT ---
    chain input {
        type filter hook input priority filter; policy drop;

        # Loopback siempre permitido
        iif lo accept

        # Conntrack: permitir tráfico de conexiones establecidas
        ct state established,related accept

        # Conntrack: tirar paquetes inválidos
        ct state invalid drop

        # Blacklist
        ip saddr @blacklist drop

        # ICMP (ping) permitido
        icmp type echo-request accept

        # ICMPv6 necesario para IPv6
        icmpv6 type { echo-request, nd-neighbor-solicit, nd-neighbor-advert, nd-router-advert } accept

        # SSH con rate limiting anti-bruteforce
        tcp dport 22 ct state new limit rate 5/minute accept

        # Web
        tcp dport { 80, 443 } ct state new accept

        # Log y drop del resto
        log prefix "INPUT-DROP: " level warn drop
    }

    # --- Cadena OUTPUT ---
    chain output {
        type filter hook output priority filter; policy accept;

        # Permitir todo saliente (política accept)
        # Agregar restricciones aquí si se necesitan
    }

    # --- Cadena FORWARD ---
    chain forward {
        type filter hook forward priority filter; policy drop;

        # Permitir forwarding de conexiones establecidas
        ct state established,related accept

        # Permitir de red interna hacia internet
        iif eth1 oif eth0 ct state new accept

        log prefix "FORWARD-DROP: " level warn drop
    }
}

# ============================================================
# TABLA NAT
# ============================================================
table ip nat {

    chain prerouting {
        type nat hook prerouting priority dstnat;

        # Port forwarding: puerto 80 externo → servidor interno
        iif eth0 tcp dport 80 dnat to 192.168.1.10:80
    }

    chain postrouting {
        type nat hook postrouting priority srcnat;

        # Masquerade para toda la red interna
        ip saddr 192.168.0.0/24 oif eth0 masquerade
    }
}
```

## Aplicar este archivo

```bash
sudo nft -f /etc/nftables.conf
```

## Verificar que se aplicó correctamente

```bash
sudo nft list ruleset
```

---

## Configuración mínima para servidor web (solo lo esencial)

```bash
sudo nft flush ruleset
sudo nft add table inet filter
sudo nft add chain inet filter input '{ type filter hook input priority 0; policy drop; }'
sudo nft add rule inet filter input iif lo accept
sudo nft add rule inet filter input ct state established,related accept
sudo nft add rule inet filter input ct state invalid drop
sudo nft add rule inet filter input tcp dport 22 limit rate 5/minute accept
sudo nft add rule inet filter input tcp dport { 80, 443 } accept
sudo nft add rule inet filter input icmp type echo-request accept
sudo nft list ruleset > /etc/nftables.conf
```

---

## Diagnóstico rápido

```bash
# ¿Hay reglas activas?
sudo nft list ruleset

# ¿Qué bloquea X conexión?
sudo nft monitor  # ver en tiempo real

# Ver logs del firewall
sudo journalctl -k | grep "INPUT-DROP"

# Probar que SSH funciona
ssh -v usuario@ip_servidor

# Probar que HTTP responde
curl -I http://ip_servidor
```
