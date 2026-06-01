# Tablas y Cadenas

## Tablas

```bash
# Agregar tabla de filtrado (inet = IPv4 + IPv6)
sudo nft add table inet filter

# Crear tabla NAT
sudo nft add table ip nat
```

## Cadenas

```bash
# Cadena básica sin política predeterminada
sudo nft add chain inet filter input '{ type filter hook input priority 0; }'

# Cadena con política DROP (bloquea todo por defecto)
sudo nft add chain inet filter input '{ type filter hook input priority 0; policy drop; }'
```

### Diferencia de políticas

| Política | Comportamiento |
|----------|----------------|
| `accept` (default) | Permite todo lo que no tenga regla explícita |
| `drop`   | Bloquea todo lo que no tenga regla explícita |
