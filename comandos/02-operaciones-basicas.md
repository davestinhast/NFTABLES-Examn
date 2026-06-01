# Operaciones Básicas

## Ver reglas

```bash
# Ver todas las reglas
sudo nft list ruleset

# Ver reglas con handles numerados
sudo nft --handle list ruleset
```

## Limpiar / Eliminar reglas

```bash
# Eliminar TODAS las reglas
sudo nft flush ruleset

# Eliminar regla específica por handle
sudo nft delete rule inet filter input handle 18
```

## Archivos de configuración

```bash
# Editar archivo principal
sudo nano /etc/nftables.conf

# Aplicar configuración desde archivo
sudo nft -f /etc/nftables.conf
```
