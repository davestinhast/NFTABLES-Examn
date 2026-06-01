# Familias, hooks y prioridades

## Familias de direccion

Cuando se crea una tabla hay que indicar con que tipo de trafico va a trabajar.
Eso se hace con la familia.

| Familia | Para que sirve |
|---------|----------------|
| `ip` | Solo paquetes IPv4 |
| `ip6` | Solo paquetes IPv6 |
| `inet` | IPv4 e IPv6 al mismo tiempo. El mas usado para firewalls |
| `arp` | Paquetes ARP (resolucion de direcciones MAC) |
| `bridge` | Paquetes que pasan por un bridge de red |
| `netdev` | Paquetes al momento de entrar o salir de una interfaz especifica |

```bash
# Tabla para IPv4 e IPv6 (la mas comun)
sudo nft add table inet mi_tabla

# Tabla solo para IPv4
sudo nft add table ip mi_tabla

# Tabla para trafico de bridge
sudo nft add table bridge mi_tabla
```

---

## Tipos de cadena

Cuando se crea una cadena hay que indicar para que se va a usar.

| Tipo | Para que sirve |
|------|----------------|
| `filter` | Para permitir o bloquear paquetes |
| `nat` | Para traduccion de direcciones (DNAT, SNAT, masquerade) |
| `route` | Para modificar como se enrutan los paquetes |

---

## Hooks - en que momento se ejecuta la cadena

El hook indica en que punto del procesamiento de red se activa la cadena.

| Hook | Cuando se ejecuta |
|------|-------------------|
| `prerouting` | Justo cuando el paquete llega, antes de decidir a donde va |
| `input` | Cuando el paquete va destinado al propio equipo |
| `forward` | Cuando el paquete va a pasar por el equipo hacia otro destino |
| `output` | Cuando el paquete fue generado por el propio equipo y esta saliendo |
| `postrouting` | Justo antes de que el paquete salga por la interfaz de red |
| `ingress` | Al entrar por una interfaz especifica (solo familia netdev) |

---

## Prioridades

La prioridad decide el orden en que se ejecutan las cadenas cuando hay varias
conectadas al mismo hook. Un numero mas bajo se ejecuta primero.

| Nombre estandar | Numero | Cuando se usa |
|-----------------|--------|---------------|
| `raw` | -300 | Antes de que conntrack registre el paquete |
| `mangle` | -150 | Para modificar campos de los paquetes |
| `dstnat` | -100 | Para DNAT en prerouting |
| `filter` | 0 | Filtrado normal, el mas comun |
| `security` | 50 | Politicas de seguridad del sistema |
| `srcnat` | 100 | Para SNAT y masquerade en postrouting |

```bash
# Usando el nombre estandar (mas legible)
sudo nft add chain inet filter input '{ type filter hook input priority filter; }'

# Usando el numero directamente
sudo nft add chain inet filter input '{ type filter hook input priority 0; }'

# Cadena de NAT que debe ejecutarse antes del filtrado
sudo nft add chain ip nat prerouting '{ type nat hook prerouting priority dstnat; }'
```

---

## Politica de cadena

La politica define que pasa con un paquete que llego al final de la cadena
sin que ninguna regla lo haya procesado.

```bash
# Policy accept - si nada coincide, se deja pasar (menos seguro)
sudo nft add chain inet filter input '{ type filter hook input priority 0; policy accept; }'

# Policy drop - si nada coincide, se bloquea (mas seguro)
sudo nft add chain inet filter input '{ type filter hook input priority 0; policy drop; }'

# Cambiar la politica de una cadena que ya existe
sudo nft chain inet filter input '{ policy drop; }'
```
