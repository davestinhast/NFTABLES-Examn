# Sets y Blacklist de IPs

Los **sets** permiten agrupar múltiples IPs para aplicar una sola regla en vez de una por cada IP.

## Crear un set de tipo blacklist

```bash
sudo nft add set inet filter blacklist { type ipv4_addr \; }
```

## Agregar una IP al blacklist

```bash
sudo nft add element inet filter blacklist { 10.0.0.5 }
```

## Bloquear todas las IPs del set

```bash
sudo nft add rule inet filter input ip saddr @blacklist drop
```

## Resultado

- Todas las IPs dentro del set `blacklist` serán rechazadas automáticamente.
- Para agregar más IPs basta con `nft add element` — la regla de bloqueo ya las cubrirá.
