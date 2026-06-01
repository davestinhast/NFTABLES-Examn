# Tablas y cadenas

## Que es una tabla

Una tabla es el contenedor principal donde viven las reglas.
Antes de crear reglas hay que tener una tabla.
Las tablas tienen una familia de direcciones que define si trabajan con IPv4, IPv6 o ambos.

## Que es una cadena

Una cadena vive dentro de una tabla y es donde se ponen las reglas una por una.
Cada cadena esta conectada a un punto del sistema (hook) para saber cuando se ejecuta.

---

## Crear una tabla

```bash
sudo nft add table inet filter
```

Crea una tabla llamada `filter` que maneja tanto IPv4 como IPv6 (eso es lo que significa `inet`).
Si se quisiera solo IPv4 se usaria `ip` en vez de `inet`.

---

## Crear una cadena basica

```bash
sudo nft add chain inet filter input '{ type filter hook input priority 0; }'
```

Crea una cadena llamada `input` dentro de la tabla `inet filter`.
Esta cadena se activa cuando llega trafico al equipo (hook input).
La prioridad 0 es la estandar para filtrado.
Como no tiene `policy`, por defecto deja pasar todo lo que no tenga regla.

## Crear una cadena con politica DROP (bloquear por defecto)

```bash
sudo nft add chain inet filter input '{ type filter hook input priority 0; policy drop; }'
```

Igual que el anterior pero con `policy drop` al final.
Esto significa que cualquier paquete que llegue y no tenga una regla que lo permita
va a ser bloqueado automaticamente.

Es la opcion mas segura porque obliga a permitir cada cosa de forma explicita.

### Diferencia entre las dos politicas

| Politica | Que pasa con lo que no tiene regla |
|----------|-------------------------------------|
| `accept` (por defecto) | Se deja pasar |
| `drop` | Se bloquea |

---

## Crear tabla para NAT

```bash
sudo nft add table ip nat
```

Crea una tabla separada llamada `nat` para manejar la traduccion de direcciones.
Aqui van las reglas de DNAT, SNAT y masquerade.
Se usa `ip` (solo IPv4) porque NAT normalmente aplica solo a IPv4.

---

## Cambiar la politica de una cadena que ya existe

```bash
sudo nft chain inet filter input '{ policy drop; }'
```

Si la cadena ya fue creada antes, este comando cambia su politica a DROP
sin necesidad de borrarla y crearla de nuevo.
