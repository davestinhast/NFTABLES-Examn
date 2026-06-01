# 🔥 NFTABLES — Guía Completa para Examen

Referencia de todos los comandos **nftables** organizados por categoría. Incluye lo básico y todo lo que suele aparecer por sorpresa en exámenes.

---

## 📋 Índice

### Fundamentos
1. [Instalación y Gestión del Servicio](./comandos/01-instalacion.md)
2. [Operaciones Básicas — list, flush, delete](./comandos/02-operaciones-basicas.md)
3. [Tablas y Cadenas](./comandos/03-tablas-y-cadenas.md)

### Reglas y Firewall
4. [Reglas de Firewall — SSH, HTTP, IP, ping](./comandos/04-reglas-firewall.md)
5. [Stateful Firewall — Conntrack ct state ⭐](./comandos/07-stateful-conntrack.md)
6. [Cadenas OUTPUT y FORWARD](./comandos/08-cadenas-output-forward.md)
7. [REJECT vs DROP y Logging](./comandos/09-reject-vs-drop-log.md)
8. [Múltiples Puertos, Rangos y CIDRs](./comandos/11-puertos-multiples-rangos-cidr.md)

### Sets y Listas
9. [Sets y Blacklist de IPs](./comandos/05-sets-blacklist.md)

### NAT
10. [NAT — DNAT, SNAT, Masquerade ⭐](./comandos/10-nat-dnat-snat-masquerade.md)

### IPv6
11. [IPv6 con nftables](./comandos/12-ipv6.md)

### Monitorización y Teoría
12. [Contadores y Monitorización](./comandos/13-contadores-monitorizacion.md)
13. [Familias, Hooks y Prioridades](./comandos/14-familias-y-prioridades.md)

### Persistencia y Ejemplos
14. [Guardar Configuración](./comandos/06-persistencia.md)
15. [Ruleset Completo de Ejemplo ⭐](./comandos/15-ruleset-completo-ejemplo.md)

> ⭐ = especialmente importante para examen

---

## ⚡ Comandos de Supervivencia (memorizar estos)

```bash
# Ver todo
sudo nft list ruleset

# Ver con handles para poder borrar
sudo nft --handle list ruleset

# Borrar todo
sudo nft flush ruleset

# Borrar regla específica
sudo nft delete rule inet filter input handle <N>

# Guardar permanente
sudo nft list ruleset > /etc/nftables.conf

# Aplicar desde archivo
sudo nft -f /etc/nftables.conf

# Monitoreo en tiempo real
sudo nft monitor
```

---

## 🗺️ Flujo Completo — Servidor Seguro

```bash
# 1. Limpiar
sudo nft flush ruleset

# 2. Tabla
sudo nft add table inet filter

# 3. Cadena INPUT con DROP
sudo nft add chain inet filter input '{ type filter hook input priority 0; policy drop; }'

# 4. Loopback
sudo nft add rule inet filter input iif lo accept

# 5. Conntrack (CRÍTICO — permite tráfico de retorno)
sudo nft add rule inet filter input ct state established,related accept
sudo nft add rule inet filter input ct state invalid drop

# 6. Servicios
sudo nft add rule inet filter input tcp dport 22 limit rate 5/minute accept
sudo nft add rule inet filter input tcp dport { 80, 443 } accept
sudo nft add rule inet filter input icmp type echo-request accept

# 7. Guardar
sudo nft list ruleset > /etc/nftables.conf
sudo systemctl enable nftables
```

---

## 📊 Tabla Resumen — Conceptos Clave

| Concepto | Comando clave |
|----------|---------------|
| Ver reglas | `nft list ruleset` |
| Borrar todo | `nft flush ruleset` |
| Política DROP | `policy drop;` en la cadena |
| Conntrack retorno | `ct state established,related accept` |
| Rate limiting | `limit rate 5/minute` |
| REJECT activo | `reject with icmp type port-unreachable` |
| Port forwarding | `dnat to IP:puerto` |
| Masquerade | `masquerade` en postrouting |
| Blacklist dinámica | `set blacklist { type ipv4_addr; }` |
| Log paquetes | `log prefix "TAG: " level warn` |
| Ver handles | `nft --handle list ruleset` |

---

## 🔑 Familias de Dirección

| Familia | Cubre |
|---------|-------|
| `ip` | Solo IPv4 |
| `ip6` | Solo IPv6 |
| `inet` | IPv4 + IPv6 juntos ✅ |
| `arp` | Paquetes ARP |
| `bridge` | Tráfico de bridge |

---

## 🪝 Hooks y Prioridades

| Hook | Cuándo | Prioridad típica |
|------|--------|-----------------|
| `prerouting` | Antes de enrutar | -100 (DNAT) |
| `input` | Hacia el sistema | 0 (filter) |
| `forward` | Atraviesa el sistema | 0 (filter) |
| `output` | Desde el sistema | 0 (filter) |
| `postrouting` | Después de enrutar | 100 (SNAT) |
