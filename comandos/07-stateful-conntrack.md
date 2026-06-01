# Firewall con seguimiento de conexiones - conntrack

## Que significa stateful

Un firewall normal revisa cada paquete por separado sin saber si es parte de
una conexion que ya estaba abierta. Un firewall stateful (con estado) si recuerda
las conexiones activas y sabe si un paquete es respuesta de algo que el propio equipo pidio.

Esto importa porque si la politica de la cadena es DROP, sin conntrack el servidor
manda una peticion hacia afuera pero la respuesta tambien queda bloqueada.

---

## Estados de conexion que maneja conntrack

| Estado | Que significa |
|--------|---------------|
| `new` | Es el primer paquete de una conexion nueva |
| `established` | La conexion ya esta activa y ambos lados se estan comunicando |
| `related` | Es una conexion nueva pero relacionada con una que ya existe (por ejemplo FTP) |
| `invalid` | El paquete no pertenece a ninguna conexion conocida o esta mal formado |

---

## Reglas basicas de conntrack

### Permitir trafico de conexiones ya abiertas

```bash
sudo nft add rule inet filter input ct state established,related accept
```

`ct state` significa "estado de la conexion segun conntrack".
Esta regla permite el paso de paquetes que son respuesta de conexiones que ya existen.
Es la regla mas importante cuando se usa politica DROP, porque sin ella
las respuestas del servidor quedarian bloqueadas.

### Tirar los paquetes que no tienen sentido

```bash
sudo nft add rule inet filter input ct state invalid drop
```

Descarta paquetes que conntrack no puede identificar como parte de ninguna conexion valida.
Estos paquetes suelen ser intentos de ataque o errores de red.

### Permitir SSH solo para conexiones nuevas y activas

```bash
sudo nft add rule inet filter input tcp dport 22 ct state new,established accept
```

Permite abrir nuevas conexiones SSH y mantener las que ya estan abiertas.

---

## Configuracion completa usando conntrack

Los pasos en orden correcto para que todo funcione:

```bash
# Paso 1 - Borrar todo para empezar limpio
sudo nft flush ruleset

# Paso 2 - Crear la tabla
sudo nft add table inet filter

# Paso 3 - Crear cadena INPUT con politica DROP
sudo nft add chain inet filter input '{ type filter hook input priority 0; policy drop; }'

# Paso 4 - Permitir loopback (trafico interno del sistema)
sudo nft add rule inet filter input iif lo accept

# Paso 5 - Permitir respuestas de conexiones que ya estaban abiertas
sudo nft add rule inet filter input ct state established,related accept

# Paso 6 - Tirar paquetes invalidos
sudo nft add rule inet filter input ct state invalid drop

# Paso 7 - Agregar los servicios que se quieren abrir
sudo nft add rule inet filter input tcp dport 22 ct state new accept
sudo nft add rule inet filter input tcp dport 80 ct state new accept
sudo nft add rule inet filter input tcp dport 443 ct state new accept
```

El orden de las reglas importa. La regla de established,related debe ir antes
de las reglas de servicios especificos para que las respuestas no queden atrapadas.
