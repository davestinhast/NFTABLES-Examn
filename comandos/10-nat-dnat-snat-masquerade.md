# NAT - DNAT, SNAT y Masquerade

## Que es NAT

NAT significa traduccion de direcciones de red. Se usa para cambiar la IP de origen
o de destino de los paquetes. Es lo que permite que varios equipos de una red interna
compartan una sola IP publica para salir a internet.

NAT necesita su propia tabla separada del filtrado.

---

## Paso 1 - Crear la tabla y las cadenas de NAT

```bash
# Crear la tabla NAT (solo IPv4)
sudo nft add table ip nat

# Crear la cadena PREROUTING para DNAT
# Se ejecuta antes de que el sistema decida a donde va el paquete
sudo nft add chain ip nat prerouting '{ type nat hook prerouting priority -100; }'

# Crear la cadena POSTROUTING para SNAT y masquerade
# Se ejecuta despues de que el sistema decide la ruta del paquete
sudo nft add chain ip nat postrouting '{ type nat hook postrouting priority 100; }'
```

---

## DNAT - Redirigir trafico entrante a otro equipo

DNAT cambia la IP de destino de un paquete. Se usa para redirigir trafico
que llega al servidor hacia otro equipo dentro de la red interna.

```bash
# Redirigir todo lo que llegue al puerto 80 hacia el servidor interno 192.168.1.10
sudo nft add rule ip nat prerouting tcp dport 80 dnat to 192.168.1.10

# Redirigir el puerto 80 externo al puerto 8080 del servidor interno
sudo nft add rule ip nat prerouting tcp dport 80 dnat to 192.168.1.10:8080

# Redirigir solo el trafico que entra por la interfaz eth0
sudo nft add rule ip nat prerouting iif eth0 tcp dport 443 dnat to 192.168.1.20:443
```

---

## SNAT - Cambiar la IP de origen de los paquetes salientes

SNAT cambia la IP de origen. Se usa cuando se conoce la IP publica fija del servidor.

```bash
# Todo lo que salga por eth0 va a aparecer como si viniera de 203.0.113.5
sudo nft add rule ip nat postrouting oif eth0 snat to 203.0.113.5

# SNAT solo para la red interna 192.168.1.0/24
sudo nft add rule ip nat postrouting ip saddr 192.168.1.0/24 oif eth0 snat to 203.0.113.5
```

---

## Masquerade - SNAT automatico cuando la IP publica cambia

Masquerade hace lo mismo que SNAT pero no necesita saber la IP de antemano.
Lee la IP actual de la interfaz de salida y la usa automaticamente.
Es el que se usa en casas o cuando la IP publica es dinamica (DHCP).

```bash
# Todo lo que salga por eth0 va a usar la IP actual de eth0 como origen
sudo nft add rule ip nat postrouting oif eth0 masquerade

# Masquerade solo para una red interna especifica
sudo nft add rule ip nat postrouting ip saddr 192.168.0.0/24 oif eth0 masquerade
```

### Diferencia rapida entre SNAT y Masquerade

| | SNAT | Masquerade |
|--|------|------------|
| IP publica | Fija, se escribe en el comando | Dinamica, la lee de la interfaz |
| Cuando usarlo | Servidores con IP estatica | Conexiones de casa o IP que cambia |

---

## Activar IP forwarding (necesario para que NAT funcione)

Sin esto el sistema no va a reenviar paquetes entre interfaces y NAT no va a funcionar.

```bash
# Activarlo ahora mismo (se pierde al reiniciar)
sudo sysctl -w net.ipv4.ip_forward=1

# Activarlo de forma permanente
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```
