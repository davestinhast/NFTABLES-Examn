# Instalación y Gestión del Servicio

## Verificar instalación

```bash
nft -version
```

## Instalar nftables

```bash
sudo apt update
sudo apt install nftables
```

## Gestión del servicio (systemctl)

```bash
# Iniciar
sudo systemctl start nftables

# Habilitar en arranque
sudo systemctl enable nftables

# Ver estado
sudo systemctl status nftables

# Reiniciar
sudo systemctl restart nftables
```
