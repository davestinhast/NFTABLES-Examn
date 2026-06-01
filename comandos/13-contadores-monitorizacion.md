# Contadores y monitoreo

## Contadores en reglas

Un contador lleva la cuenta de cuantos paquetes y cuantos bytes pasaron por una regla.
Es util para saber si una regla esta siendo usada o no, y cuanto trafico esta procesando.

### Agregar un contador a una regla

```bash
sudo nft add rule inet filter input tcp dport 80 counter accept
```

Igual que antes pero con `counter` en el medio. Cada vez que un paquete pasa por
esta regla el contador sube. Al correr `nft list ruleset` se puede ver cuantos paquetes pasaron:

```
tcp dport 80 counter packets 1523 bytes 98432 accept
```

### Contador con nombre (se puede consultar por separado)

```bash
# Paso 1 - Crear el contador con un nombre
sudo nft add counter inet filter trafico_http

# Paso 2 - Usar ese contador en la regla
sudo nft add rule inet filter input tcp dport 80 counter name trafico_http accept

# Ver solo ese contador
sudo nft list counter inet filter trafico_http

# Ver todos los contadores
sudo nft list counters
```

---

## nft monitor - ver que pasa en tiempo real

```bash
# Ver todos los cambios: tablas, cadenas y reglas que se agreguen o borren
sudo nft monitor

# Ver solo cambios en reglas
sudo nft monitor rules

# Ver solo cambios en tablas
sudo nft monitor tables
```

`nft monitor` es muy util cuando se esta configurando el firewall y se quiere ver
en tiempo real que reglas se estan aplicando sin tener que correr `list ruleset` cada vez.
Para salir hay que presionar Ctrl+C.

---

## Cuotas - limitar por cantidad de datos

```bash
# Bloquear a una IP despues de que transfiera mas de 1 GB
sudo nft add rule inet filter input ip saddr 10.0.0.5 quota over 1 gbytes drop

# Limitar la descarga total del servidor a 500 MB
sudo nft add rule inet filter output quota over 500 mbytes drop
```

---

## Ver las interfaces de red disponibles

Esto no es de nftables pero es necesario para saber que nombres poner en las reglas
que usan `iif` (interfaz de entrada) o `oif` (interfaz de salida).

```bash
# Ver todas las interfaces de red del sistema
ip link show

# Tambien se puede usar
nmcli device status
```

Una vez que se sabe el nombre de la interfaz (por ejemplo eth0 o wlan0) se usa asi:

```bash
# Permitir solo lo que llega por eth0
sudo nft add rule inet filter input iif "eth0" accept

# Bloquear todo lo que sale por wlan0
sudo nft add rule inet filter output oif "wlan0" drop
```
