# Varios puertos, rangos y bloques de IPs

## Varios puertos en una sola regla

En vez de crear una regla por cada puerto, se pueden poner varios dentro de llaves
separados por comas. Esto hace lo mismo pero con un solo comando.

```bash
# Permitir SSH, HTTP y HTTPS con una sola regla
sudo nft add rule inet filter input tcp dport { 22, 80, 443 } accept

# Bloquear puertos inseguros conocidos
sudo nft add rule inet filter input tcp dport { 23, 135, 139, 445 } drop

# Permitir varios puertos UDP
sudo nft add rule inet filter input udp dport { 53, 67, 68 } accept
```

---

## Rangos de puertos

Se pueden especificar rangos de puertos con un guion entre el primero y el ultimo.

```bash
# Permitir cualquier puerto entre 8000 y 9000
sudo nft add rule inet filter input tcp dport 8000-9000 accept

# Bloquear todos los puertos altos entrantes
sudo nft add rule inet filter input tcp dport 1024-65535 drop
```

---

## Bloques de IPs con notacion CIDR

En vez de escribir cada IP de una red, se puede usar la notacion CIDR
que representa un bloque completo de IPs.

Por ejemplo `192.168.1.0/24` representa todas las IPs desde 192.168.1.0 hasta 192.168.1.255.

```bash
# Permitir toda una red local
sudo nft add rule inet filter input ip saddr 192.168.1.0/24 accept

# Bloquear un rango completo de IPs sospechosas
sudo nft add rule inet filter input ip saddr 10.0.0.0/8 drop

# Solo permitir acceso a SSH desde la red de administracion
sudo nft add rule inet filter input ip saddr 172.16.0.0/12 tcp dport 22 accept
```

---

## Combinar IP y puerto en la misma regla

```bash
# Solo la IP 192.168.1.50 puede acceder a MySQL (puerto 3306)
sudo nft add rule inet filter input ip saddr 192.168.1.50 tcp dport 3306 accept

# Bloquear acceso a SSH desde toda una subred
sudo nft add rule inet filter input ip saddr 10.10.0.0/16 tcp dport 22 drop
```

---

## Sets con bloques CIDR (interval sets)

Para manejar varios bloques de IPs con una sola regla hay que crear un set
con el flag `interval` que permite guardar rangos en vez de solo IPs sueltas.

```bash
# Paso 1 - Crear el set con soporte para intervalos
sudo nft add set inet filter redes_confiables { type ipv4_addr\; flags interval\; }

# Paso 2 - Agregar bloques de IPs al set
sudo nft add element inet filter redes_confiables { 192.168.0.0/24, 10.0.0.0/8 }

# Paso 3 - Crear la regla que usa el set
sudo nft add rule inet filter input ip saddr @redes_confiables accept
```

El resultado es que cualquier IP dentro de esas dos redes va a ser permitida
con una sola regla en vez de dos reglas separadas.
