# Ejemplo de configuracion completa

Este es un ejemplo real y funcional de como se veria un servidor configurado
con nftables cubriendo todos los casos importantes. Esta escrito en formato
de archivo para guardar en /etc/nftables.conf.

---

## Configuracion completa como archivo

```bash
#!/usr/sbin/nft -f

# Borrar todo lo que haya antes para empezar desde cero
flush ruleset

# ============================================================
# TABLA DE FILTRADO - cubre IPv4 e IPv6
# ============================================================
table inet filter {

    # Lista negra de IPs bloqueadas
    # Se pueden agregar IPs o rangos completos
    set blacklist {
        type ipv4_addr
        flags interval
        elements = { 10.0.0.5, 192.168.50.0/24 }
    }

    # Cadena que maneja el trafico entrante
    chain input {
        type filter hook input priority filter; policy drop;

        # El loopback siempre debe estar permitido
        iif lo accept

        # Permitir trafico de conexiones que ya estaban abiertas
        # Sin esta regla las respuestas del servidor quedarian bloqueadas
        ct state established,related accept

        # Tirar paquetes mal formados o invalidos
        ct state invalid drop

        # Bloquear todo lo que este en la lista negra
        ip saddr @blacklist drop

        # Permitir ping
        icmp type echo-request accept

        # Permitir los mensajes ICMPv6 que IPv6 necesita para funcionar
        icmpv6 type { echo-request, nd-neighbor-solicit, nd-neighbor-advert, nd-router-advert } accept

        # Permitir SSH con limite de 5 intentos por minuto
        tcp dport 22 ct state new limit rate 5/minute accept

        # Permitir trafico web
        tcp dport { 80, 443 } ct state new accept

        # Registrar y bloquear todo lo que llegue hasta aca sin ser aceptado
        log prefix "INPUT-DROP: " level warn drop
    }

    # Cadena para el trafico saliente
    chain output {
        type filter hook output priority filter; policy accept;

        # Con policy accept todo lo que salga esta permitido por defecto
        # Si se quiere restringir la salida se puede cambiar a policy drop
        # y agregar reglas especificas aqui
    }

    # Cadena para trafico que pasa por el equipo (para cuando actua de router)
    chain forward {
        type filter hook forward priority filter; policy drop;

        # Permitir conexiones activas que estan pasando por el equipo
        ct state established,related accept

        # Permitir trafico desde la red interna (eth1) hacia internet (eth0)
        iif eth1 oif eth0 ct state new accept

        # Registrar y bloquear el resto
        log prefix "FORWARD-DROP: " level warn drop
    }
}

# ============================================================
# TABLA NAT - solo IPv4
# ============================================================
table ip nat {

    chain prerouting {
        type nat hook prerouting priority dstnat;

        # Redirigir el puerto 80 que llega por eth0 al servidor interno
        iif eth0 tcp dport 80 dnat to 192.168.1.10:80
    }

    chain postrouting {
        type nat hook postrouting priority srcnat;

        # Compartir la IP de eth0 con toda la red interna
        ip saddr 192.168.0.0/24 oif eth0 masquerade
    }
}
```

---

## Como aplicar esta configuracion

```bash
# Guardar el contenido de arriba en el archivo de configuracion
sudo nano /etc/nftables.conf

# Aplicarlo
sudo nft -f /etc/nftables.conf

# Verificar que se cargo correctamente
sudo nft list ruleset
```

---

## Configuracion minima para un servidor web

Para quien solo necesita lo basico sin NAT ni forward:

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
sudo systemctl enable nftables
```

---

## Comandos para revisar que todo funciona

```bash
# Ver todas las reglas activas
sudo nft list ruleset

# Ver que esta siendo bloqueado en tiempo real
sudo nft monitor

# Buscar en los logs lo que fue bloqueado
sudo journalctl -k | grep "INPUT-DROP"

# Probar que SSH responde
ssh -v usuario@ip_del_servidor

# Probar que el servidor web responde
curl -I http://ip_del_servidor
```
