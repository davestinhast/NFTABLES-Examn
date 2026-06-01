# Familias de Dirección y Prioridades de Cadena

## Familias disponibles en nftables

| Familia | Descripción |
|---------|-------------|
| `ip` | Paquetes IPv4 |
| `ip6` | Paquetes IPv6 |
| `inet` | IPv4 + IPv6 (recomendado para la mayoría de casos) |
| `arp` | Paquetes ARP |
| `bridge` | Paquetes que atraviesan un bridge |
| `netdev` | Ingress/egress de interfaz específica |

```bash
# Tabla ip (solo IPv4)
sudo nft add table ip mi_tabla

# Tabla inet (ambos)
sudo nft add table inet mi_tabla

# Tabla para bridge
sudo nft add table bridge mi_tabla
```

---

## Tipos de cadena (type)

| Tipo | Uso |
|------|-----|
| `filter` | Filtrado de paquetes (accept/drop) |
| `nat` | Traducción de direcciones |
| `route` | Modificar enrutamiento |

---

## Hooks disponibles

| Hook | Cuándo se ejecuta |
|------|-------------------|
| `prerouting` | Antes de la decisión de enrutamiento |
| `input` | Paquetes destinados al sistema local |
| `forward` | Paquetes que atraviesan el sistema |
| `output` | Paquetes generados localmente |
| `postrouting` | Después de la decisión de enrutamiento |
| `ingress` | Al llegar a la interfaz (familia netdev) |

---

## Prioridades de cadena

Número **menor** = se ejecuta **antes**. Puede ser negativo.

| Nombre estándar | Valor | Uso típico |
|-----------------|-------|-----------|
| `raw` | -300 | Antes de conntrack |
| `mangle` | -150 | Modificar campos de paquetes |
| `dstnat` | -100 | DNAT / prerouting NAT |
| `filter` | 0 | Filtrado estándar |
| `security` | 50 | Políticas SELinux/AppArmor |
| `srcnat` | 100 | SNAT / postrouting NAT |

```bash
# Usando nombre estándar
sudo nft add chain inet filter input '{ type filter hook input priority filter; }'

# Usando número directamente
sudo nft add chain inet filter input '{ type filter hook input priority 0; }'

# NAT prerouting (debe ser antes del filtrado)
sudo nft add chain ip nat prerouting '{ type nat hook prerouting priority dstnat; }'
```

---

## Política predeterminada de cadena

```bash
# Policy ACCEPT (permite todo lo no especificado)
sudo nft add chain inet filter input '{ type filter hook input priority 0; policy accept; }'

# Policy DROP (bloquea todo lo no especificado — más seguro)
sudo nft add chain inet filter input '{ type filter hook input priority 0; policy drop; }'

# Cambiar política de cadena existente
sudo nft chain inet filter input '{ policy drop; }'
```
