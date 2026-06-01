# Sets y lista negra de IPs

## Que es un set

Un set es una lista donde se guardan varias IPs juntas.
En vez de crear una regla de bloqueo por cada IP, se crea un set con todas las IPs
y una sola regla que bloquea todo lo que este en ese set.
Es mucho mas comodo cuando hay muchas IPs que manejar.

---

## Paso 1 - Crear el set vacio

```bash
sudo nft add set inet filter blacklist { type ipv4_addr \; }
```

Crea un set llamado `blacklist` dentro de la tabla `inet filter`.
`type ipv4_addr` indica que en este set van a guardarse direcciones IPv4.
El set empieza vacio, todavia no bloquea nada.

## Paso 2 - Agregar una IP al set

```bash
sudo nft add element inet filter blacklist { 10.0.0.5 }
```

Agrega la IP 10.0.0.5 al set blacklist.
Se pueden agregar mas IPs en el mismo comando separandolas con comas:

```bash
sudo nft add element inet filter blacklist { 10.0.0.5, 10.0.0.6, 10.0.0.7 }
```

Agregar una IP al set no la bloquea todavia. Eso lo hace la siguiente regla.

## Paso 3 - Crear la regla que bloquea todo el set

```bash
sudo nft add rule inet filter input ip saddr @blacklist drop
```

`ip saddr @blacklist` significa "si la IP de origen esta dentro del set llamado blacklist".
Esta regla revisa cada paquete que llega y si la IP esta en la lista negra, lo descarta.
El `@` antes del nombre indica que se esta referenciando un set.

---

## Para ver que hay en el set

```bash
sudo nft list set inet filter blacklist
```

Muestra las IPs que estan guardadas en el set en este momento.

## Para eliminar una IP del set

```bash
sudo nft delete element inet filter blacklist { 10.0.0.5 }
```

Saca la IP 10.0.0.5 de la lista. La regla de bloqueo sigue existiendo
pero ya no le aplica a esa IP.
