# REJECT vs DROP y como registrar paquetes

## La diferencia entre DROP y REJECT

Ambos bloquean el paquete, pero la diferencia esta en lo que le dicen al que envio el paquete.

| Comando | Que hace | Lo que siente el cliente |
|---------|----------|--------------------------|
| `drop` | Tira el paquete en silencio y no dice nada | El cliente espera hasta que el intento expira, puede tardar varios segundos |
| `reject` | Bloquea el paquete pero manda un mensaje de error de vuelta | El cliente recibe el error de inmediato y sabe que fue rechazado |

### Cuando usar cada uno

- `drop` se usa contra atacantes o trafico no deseado. El silencio hace que no sepan si el puerto existe.
- `reject` se usa cuando hay servicios internos y se quiere que el error aparezca rapido.

---

## Comandos DROP

```bash
# Bloquear silenciosamente todo lo que llegue al puerto 23 (telnet)
sudo nft add rule inet filter input tcp dport 23 drop
```

---

## Comandos REJECT

```bash
# Rechazar con mensaje ICMP de puerto no disponible
sudo nft add rule inet filter input tcp dport 23 reject with icmp type port-unreachable

# Rechazar con un reset TCP (mas limpio para conexiones TCP)
sudo nft add rule inet filter input tcp dport 23 reject with tcp reset

# Rechazar en IPv6 con mensaje ICMPv6
sudo nft add rule inet6 filter input tcp dport 23 reject with icmpv6 type port-unreachable
```

---

## Como registrar paquetes (logging)

El logging guarda informacion de los paquetes en el log del sistema.
Sirve para ver que esta siendo bloqueado y para diagnosticar problemas.

### Registrar y despues bloquear

```bash
sudo nft add rule inet filter input ip saddr 10.0.0.5 log drop
```

Primero escribe en el log que llego un paquete de esa IP y luego lo descarta.

### Registrar con un texto para identificarlo facil

```bash
sudo nft add rule inet filter input tcp dport 22 log prefix "SSH-INTENTO: " level info accept
```

`prefix` agrega un texto al inicio de cada linea del log para que sea facil buscarla.
Cuando alguien se conecte por SSH va a aparecer una linea con "SSH-INTENTO:" en el log.

### Registrar todo lo que se esta bloqueando

```bash
sudo nft add rule inet filter input log prefix "BLOQUEADO: " level warn drop
```

Esta regla va al final de la cadena. Cualquier paquete que llego hasta aca
y no fue aceptado por ninguna regla anterior queda registrado y luego se descarta.

---

## Niveles de log disponibles

| Nivel | Para que se usa |
|-------|-----------------|
| `debug` | Para pruebas y desarrollo, muy detallado |
| `info` | Informacion general, eventos normales |
| `notice` | Algo que vale la pena saber pero no es un problema |
| `warn` | Algo que puede ser un problema |
| `err` | Error |
| `crit` | Error critico |
| `alert` | Hay que actuar ahora |
| `emerg` | El sistema esta inutilizable |

---

## Como ver los logs generados

```bash
# Ver logs del kernel que incluyan el prefijo configurado
sudo journalctl -k | grep "BLOQUEADO"

# O con dmesg
sudo dmesg | grep "BLOQUEADO"
```
