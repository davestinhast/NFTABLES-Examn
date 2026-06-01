# Guardar y Persistir Configuración

## Guardar reglas actuales

```bash
sudo nft list ruleset > /etc/nftables.conf
```
> Las reglas persistirán tras reiniciar el sistema.

## Aplicar reglas guardadas manualmente

```bash
sudo nft -f /etc/nftables.conf
```

## Flujo recomendado

```bash
# 1. Crear/modificar reglas en vivo
sudo nft add rule inet filter input tcp dport 8080 accept

# 2. Verificar que todo está bien
sudo nft list ruleset

# 3. Guardar para persistencia
sudo nft list ruleset > /etc/nftables.conf

# 4. Habilitar servicio para que cargue al inicio
sudo systemctl enable nftables
```
