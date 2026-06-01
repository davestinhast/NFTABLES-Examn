# Cadenas OUTPUT y FORWARD

## Las tres cadenas principales

| Cadena | Para que sirve |
|--------|----------------|
| `input` | Maneja el trafico que llega al equipo |
| `output` | Maneja el trafico que sale del equipo |
| `forward` | Maneja el trafico que pasa por el equipo hacia otro destino |

La cadena `input` es la mas comun. Las cadenas `output` y `forward` se usan cuando
se necesita controlar lo que el equipo envia o cuando el equipo actua como router.

---

## Cadena OUTPUT

### Crear la cadena de salida

```bash
sudo nft add chain inet filter output '{ type filter hook output priority 0; policy accept; }'
```

Crea la cadena que controla el trafico que sale del equipo.
Con `policy accept` todo lo que salga esta permitido por defecto.

### Bloquear trafico saliente hacia una IP

```bash
sudo nft add rule inet filter output ip daddr 10.0.0.5 drop
```

`ip daddr` significa "IP de destino".
Este comando impide que el equipo se conecte a la IP 10.0.0.5.

### Permitir solo salida por HTTP y HTTPS

```bash
sudo nft add rule inet filter output tcp dport { 80, 443 } accept
```

Si la politica de output es DROP, este comando permite que el equipo
se conecte a servidores web pero no a otros puertos.

### Cadena de salida con politica restrictiva

```bash
# Primero crear la cadena con politica DROP
sudo nft add chain inet filter output '{ type filter hook output priority 0; policy drop; }'

# Permitir respuestas de conexiones activas
sudo nft add rule inet filter output ct state established,related accept

# Permitir salida web y DNS
sudo nft add rule inet filter output tcp dport { 80, 443, 53 } accept
```

---

## Cadena FORWARD

La cadena FORWARD se usa cuando el equipo actua como router o gateway,
es decir cuando los paquetes entran por una interfaz y salen por otra.

### Crear la cadena forward

```bash
sudo nft add chain inet filter forward '{ type filter hook forward priority 0; policy drop; }'
```

Crea la cadena con politica DROP. Todo lo que quiera pasar por el equipo
sera bloqueado a menos que haya una regla que lo permita.

### Permitir trafico de conexiones activas que estan pasando

```bash
sudo nft add rule inet filter forward ct state established,related accept
```

Permite que las conexiones que ya estaban abiertas sigan pasando por el equipo.

### Permitir que la red interna salga a internet

```bash
sudo nft add rule inet filter forward iif eth1 oif eth0 accept
```

`iif` es la interfaz de entrada y `oif` es la interfaz de salida.
Este comando permite que el trafico que entra por eth1 (red interna) salga por eth0 (internet).

### Bloquear que internet llegue a la red interna

```bash
sudo nft add rule inet filter forward iif eth0 oif eth1 ct state new drop
```

Bloquea conexiones nuevas que vengan de internet (eth0) hacia la red interna (eth1).
Las conexiones que ya estaban activas siguen funcionando por la regla anterior.
