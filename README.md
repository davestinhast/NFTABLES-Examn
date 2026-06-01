# 🔥 NFTABLES - Guía de Comandos para Examen

Referencia rápida de todos los comandos **nftables** organizados por categoría.

---

## 📋 Índice

1. [Instalación y Verificación](#instalación-y-verificación)
2. [Gestión del Servicio](#gestión-del-servicio)
3. [Operaciones Básicas de Reglas](#operaciones-básicas-de-reglas)
4. [Tablas y Cadenas](#tablas-y-cadenas)
5. [Reglas de Firewall](#reglas-de-firewall)
6. [NAT](#nat)
7. [Rate Limiting](#rate-limiting)
8. [Sets / Blacklist de IPs](#sets--blacklist-de-ips)
9. [Guardar Configuración](#guardar-configuración)

---

## Instalación y Verificación

```bash
# Verificar versión instalada
nft -version

# Instalar nftables
sudo apt update
sudo apt install nftables
```

---

## Gestión del Servicio

```bash
# Iniciar el servicio
sudo systemctl start nftables

# Habilitar al inicio del sistema
sudo systemctl enable nftables

# Verificar estado
sudo systemctl status nftables

# Reiniciar el servicio
sudo systemctl restart nftables
```

---

## Operaciones Básicas de Reglas

```bash
# Ver todas las reglas actuales
sudo nft list ruleset

# Ver reglas con handles numerados (para poder eliminarlas)
sudo nft --handle list ruleset

# Eliminar TODAS las reglas (flush completo)
sudo nft flush ruleset

# Eliminar una regla específica por handle
sudo nft delete rule inet filter input handle 18

# Editar el archivo de configuración principal
sudo nano /etc/nftables.conf

# Aplicar configuración desde archivo
sudo nft -f /etc/nftables.conf
```

---

## Tablas y Cadenas

```bash
# Agregar una tabla
sudo nft add table inet filter

# Crear una cadena básica (INPUT)
sudo nft add chain inet filter input '{ type filter hook input priority 0; }'

# Crear cadena INPUT con política DROP (bloquea todo por defecto)
sudo nft add chain inet filter input '{ type filter hook input priority 0; policy drop; }'
```

> ⚠️ Con `policy drop`: todo tráfico entrante será **bloqueado** hasta que agregues reglas de permiso explícitas.

---

## Reglas de Firewall

### Permitir tráfico local (loopback)
```bash
sudo nft add rule inet filter input iif lo accept
```
- Permite comunicación interna del sistema (localhost).
- Las aplicaciones locales funcionarán correctamente.

### Permitir SSH (puerto 22)
```bash
sudo nft add rule inet filter input tcp dport 22 accept
```
- Permite acceso remoto SSH.

### Limitar conexiones SSH (anti-bruteforce)
```bash
sudo nft add rule inet filter input tcp dport 22 limit rate 5/minute accept
```
- Máximo 5 conexiones por minuto — protege contra ataques de fuerza bruta.

### Permitir HTTP (puerto 80)
```bash
sudo nft add rule inet filter input tcp dport 80 accept
```
- El servidor web responderá en puerto 80.

### Permitir HTTPS (puerto 443)
```bash
sudo nft add rule inet filter input tcp dport 443 accept
```
- El sitio web funcionará con SSL/TLS.

### Bloquear una IP específica
```bash
sudo nft add rule inet filter input ip saddr 192.168.1.100 drop
```
- La IP `192.168.1.100` no podrá conectarse al servidor.

### Permitir ping (ICMP echo-request)
```bash
sudo nft add rule inet filter input icmp type echo-request accept
```
- El equipo responderá a ping.

### Bloquear ping (ICMP echo-request)
```bash
sudo nft add rule inet filter input icmp type echo-request drop
```
- El servidor será invisible a ping.

---

## NAT

```bash
# Crear tabla NAT
sudo nft add table ip nat
```
- Crea tabla para traducción de direcciones (NAT).
- Se podrán crear reglas NAT después.

---

## Rate Limiting

```bash
# SSH con límite de tasa (5 conexiones/minuto)
sudo nft add rule inet filter input tcp dport 22 limit rate 5/minute accept
```

---

## Sets / Blacklist de IPs

```bash
# Crear conjunto (set) de IPs bloqueadas
sudo nft add set inet filter blacklist { type ipv4_addr \; }

# Agregar una IP al blacklist
sudo nft add element inet filter blacklist { 10.0.0.5 }

# Bloquear todas las IPs del blacklist
sudo nft add rule inet filter input ip saddr @blacklist drop
```

> Con sets puedes administrar múltiples IPs de forma centralizada sin crear una regla por cada una.

---

## Guardar Configuración

```bash
# Guardar reglas actuales al archivo de configuración (persistencia)
sudo nft list ruleset > /etc/nftables.conf
```
- Las reglas **persistirán** tras reiniciar el sistema.

---

## 🗺️ Flujo Rápido — Configuración Completa de Firewall

```bash
# 1. Limpiar reglas anteriores
sudo nft flush ruleset

# 2. Crear tabla
sudo nft add table inet filter

# 3. Crear cadena INPUT con política DROP
sudo nft add chain inet filter input '{ type filter hook input priority 0; policy drop; }'

# 4. Permitir loopback
sudo nft add rule inet filter input iif lo accept

# 5. Permitir SSH (con rate limiting)
sudo nft add rule inet filter input tcp dport 22 limit rate 5/minute accept

# 6. Permitir HTTP y HTTPS
sudo nft add rule inet filter input tcp dport 80 accept
sudo nft add rule inet filter input tcp dport 443 accept

# 7. Guardar
sudo nft list ruleset > /etc/nftables.conf
```

---

> 📁 Ver también: [`comandos/`](./comandos/) para archivos por categoría.
