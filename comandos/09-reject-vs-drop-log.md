# REJECT vs DROP — y Logging

## DROP vs REJECT

| Comando | Comportamiento | Efecto en el cliente |
|---------|---------------|----------------------|
| `drop`  | Descarta silenciosamente el paquete | El cliente espera timeout (lento) |
| `reject` | Envía respuesta de rechazo activo | El cliente recibe error inmediato |

```bash
# DROP silencioso (el cliente no sabe qué pasó)
sudo nft add rule inet filter input tcp dport 23 drop

# REJECT con mensaje ICMP port-unreachable
sudo nft add rule inet filter input tcp dport 23 reject with icmp type port-unreachable

# REJECT TCP con RST (más limpio para TCP)
sudo nft add rule inet filter input tcp dport 23 reject with tcp reset

# REJECT para IPv6
sudo nft add rule inet6 filter input tcp dport 23 reject with icmpv6 type port-unreachable
```

> **¿Cuándo usar cuál?**
> - `drop` → para atacantes (no revela información, fuerza timeout)
> - `reject` → para servicios internos o cuando la UX importa

---

## Logging de paquetes

```bash
# Log básico antes de una regla
sudo nft add rule inet filter input ip saddr 10.0.0.5 log drop

# Log con prefijo descriptivo
sudo nft add rule inet filter input tcp dport 22 log prefix "SSH-INTENTO: " level info accept

# Log de paquetes dropeados (política final)
sudo nft add rule inet filter input log prefix "DROPPED: " level warn drop

# Log con contador de bytes/paquetes
sudo nft add rule inet filter input tcp dport 80 counter log prefix "HTTP: " accept
```

### Niveles de log disponibles

| Nivel | Uso |
|-------|-----|
| `emerg` | Sistema inutilizable |
| `alert` | Acción inmediata requerida |
| `crit` | Condición crítica |
| `err` | Error |
| `warn` | Advertencia |
| `notice` | Condición normal pero significativa |
| `info` | Informativo |
| `debug` | Debug |

```bash
# Ver logs generados
sudo journalctl -k | grep "SSH-INTENTO"
# o
sudo dmesg | grep "DROPPED"
```
