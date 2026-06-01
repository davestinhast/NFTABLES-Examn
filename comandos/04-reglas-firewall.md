# Reglas de firewall comunes

## Permitir trafico del propio sistema (loopback)

```bash
sudo nft add rule inet filter input iif lo accept
```

`iif lo` significa "interfaz de entrada: lo", que es la interfaz loopback.
Esto permite que el sistema hable consigo mismo.
Es necesario para que funcionen bases de datos locales, servicios internos y otras cosas.
Siempre se debe tener esta regla cuando la politica de la cadena es DROP.

---

## Permitir SSH (puerto 22)

```bash
sudo nft add rule inet filter input tcp dport 22 accept
```

Permite que alguien se conecte al equipo por SSH.
`tcp dport 22` significa "trafico TCP que llega al puerto 22".
Sin esta regla no se puede acceder al servidor de forma remota.

---

## Permitir HTTP (puerto 80)

```bash
sudo nft add rule inet filter input tcp dport 80 accept
```

Permite conexiones web sin cifrar.
Un servidor web como Apache o Nginx necesita este puerto abierto para responder.

---

## Permitir HTTPS (puerto 443)

```bash
sudo nft add rule inet filter input tcp dport 443 accept
```

Permite conexiones web con SSL/TLS (las seguras, con el candado en el navegador).
Se debe tener este puerto abierto ademas del 80 si el servidor usa certificado SSL.

---

## Bloquear una IP en especifico

```bash
sudo nft add rule inet filter input ip saddr 192.168.1.100 drop
```

`ip saddr` significa "direccion IP de origen".
Este comando bloquea todo el trafico que venga de la IP 192.168.1.100.
Esa IP no va a poder conectarse a ningún servicio del equipo.

---

## Permitir ping

```bash
sudo nft add rule inet filter input icmp type echo-request accept
```

Permite que otros equipos hagan ping al equipo.
`icmp type echo-request` es el tipo de paquete que se manda cuando se hace ping.
Si esta regla no existe y la politica es DROP, el equipo no va a responder a ping.

## Bloquear ping

```bash
sudo nft add rule inet filter input icmp type echo-request drop
```

Hace que el equipo no responda a ping.
Util para que el servidor no sea visible a escaneos basicos de red.

---

## Limitar intentos de conexion SSH (anti ataques)

```bash
sudo nft add rule inet filter input tcp dport 22 limit rate 5/minute accept
```

Permite SSH pero con un limite de 5 conexiones nuevas por minuto.
Si alguien intenta conectarse mas de 5 veces por minuto, las conexiones extras se bloquean.
Esto protege contra ataques que intentan adivinar la contrasena probando muchas veces.
