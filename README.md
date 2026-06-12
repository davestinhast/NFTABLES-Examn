# NFTABLES - Guia de Comandos para Examen

Esta guia tiene todos los comandos de nftables organizados por tema.
Cada archivo explica que hace cada comando y para que sirve.


## Indice

### Lo basico
1. [Instalacion y manejo del servicio](./comandos/01-instalacion.md)
2. [Ver, borrar y editar reglas](./comandos/02-operaciones-basicas.md)
3. [Tablas y cadenas](./comandos/03-tablas-y-cadenas.md)

### Reglas de firewall
4. [Reglas comunes - SSH, HTTP, bloquear IPs, ping](./comandos/04-reglas-firewall.md)
5. [Firewall con seguimiento de conexiones - conntrack](./comandos/07-stateful-conntrack.md)
6. [Cadenas OUTPUT y FORWARD](./comandos/08-cadenas-output-forward.md)
7. [REJECT vs DROP y como registrar paquetes](./comandos/09-reject-vs-drop-log.md)
8. [Varios puertos, rangos y bloques de IPs](./comandos/11-puertos-multiples-rangos-cidr.md)

### Listas de IPs
9. [Sets y lista negra de IPs](./comandos/05-sets-blacklist.md)

### NAT
10. [NAT - DNAT, SNAT y Masquerade](./comandos/10-nat-dnat-snat-masquerade.md)

### IPv6
11. [IPv6 con nftables](./comandos/12-ipv6.md)

### Ver que pasa y teoria
12. [Contadores y monitoreo](./comandos/13-contadores-monitorizacion.md)
13. [Familias, hooks y prioridades](./comandos/14-familias-y-prioridades.md)

### Guardar y ejemplos
14. [Guardar la configuracion](./comandos/06-persistencia.md)
15. [Ejemplo de configuracion completa](./comandos/15-ruleset-completo-ejemplo.md)


## Comandos que hay que saber si o si

```bash
# Ver todas las reglas que estan activas ahora mismo
sudo nft list ruleset

# Ver las reglas con su numero (handle), necesario para borrar una en especifico
sudo nft --handle list ruleset

# Borrar todas las reglas de un golpe
sudo nft flush ruleset

# Borrar una regla en especifico usando su numero
sudo nft delete rule inet filter input handle <N>

# Guardar las reglas para que no se pierdan al reiniciar
sudo nft list ruleset > /etc/nftables.conf

# Aplicar un archivo de configuracion
sudo nft -f /etc/nftables.conf

# Ver en tiempo real que paquetes estan siendo procesados
sudo nft monitor
```


## Como armar un firewall basico paso a paso

```bash
# Paso 1 - Borrar todo lo que haya antes para empezar limpio
sudo nft flush ruleset

# Paso 2 - Crear la tabla principal donde van a vivir las reglas
sudo nft add table inet filter

# Paso 3 - Crear la cadena INPUT con politica DROP
# Esto significa que todo lo que llegue y no tenga una regla que lo permita, se bloquea
sudo nft add chain inet filter input '{ type filter hook input priority 0; policy drop; }'

# Paso 4 - Permitir el trafico interno del propio sistema (loopback)
# Sin esto muchas cosas del sistema dejan de funcionar
sudo nft add rule inet filter input iif lo accept

# Paso 5 - Permitir el trafico de conexiones que ya estaban abiertas
# Esto es importante porque sin esta regla las respuestas del servidor tambien se bloquean
sudo nft add rule inet filter input ct state established,related accept

# Paso 6 - Tirar los paquetes que no tienen sentido o estan mal formados
sudo nft add rule inet filter input ct state invalid drop

# Paso 7 - Permitir SSH con limite de intentos para evitar ataques
sudo nft add rule inet filter input tcp dport 22 limit rate 5/minute accept

# Paso 8 - Permitir trafico web
sudo nft add rule inet filter input tcp dport { 80, 443 } accept

# Paso 9 - Permitir ping
sudo nft add rule inet filter input icmp type echo-request accept

# Paso 10 - Guardar todo para que sobreviva a un reinicio
sudo nft list ruleset > /etc/nftables.conf
sudo systemctl enable nftables
```


## Tabla resumen - lo que cae en examenes

| Concepto | Comando |
|----------|---------|
| Ver reglas | `nft list ruleset` |
| Borrar todo | `nft flush ruleset` |
| Bloquear por defecto | `policy drop;` en la cadena |
| Permitir respuestas | `ct state established,related accept` |
| Limitar intentos | `limit rate 5/minute` |
| Rechazar con aviso | `reject with icmp type port-unreachable` |
| Redirigir puertos | `dnat to IP:puerto` |
| Compartir internet | `masquerade` en postrouting |
| Lista negra de IPs | `set blacklist { type ipv4_addr; }` |
| Registrar paquetes | `log prefix "TAG: " level warn` |
| Ver numeros de regla | `nft --handle list ruleset` |


## Familias de direccion

| Familia | Para que sirve |
|---------|----------------|
| `ip` | Solo IPv4 |
| `ip6` | Solo IPv6 |
| `inet` | IPv4 e IPv6 al mismo tiempo, el mas usado |
| `arp` | Paquetes ARP |
| `bridge` | Trafico que pasa por un bridge |


## Hooks y en que momento se ejecutan

| Hook | Cuando se ejecuta | Prioridad normal |
|------|-------------------|-----------------|
| `prerouting` | Antes de decidir hacia donde va el paquete | -100 para DNAT |
| `input` | Paquetes que van al propio equipo | 0 |
| `forward` | Paquetes que pasan por el equipo hacia otro lado | 0 |
| `output` | Paquetes que salen del propio equipo | 0 |
| `postrouting` | Despues de decidir la ruta del paquete | 100 para SNAT |
