# Instalacion y manejo del servicio

## Ver si nftables ya esta instalado

```bash
nft -version
```

Esto muestra la version instalada. Si sale un numero de version, ya esta listo.
Si da error de comando no encontrado, hay que instalarlo.

## Instalar nftables

```bash
# Primero actualizar la lista de paquetes disponibles
sudo apt update

# Despues instalar nftables
sudo apt install nftables
```

Hay que correr los dos comandos en orden. El primero actualiza la lista de paquetes,
el segundo descarga e instala nftables.

---

## Manejar el servicio con systemctl

### Iniciar el servicio

```bash
sudo systemctl start nftables
```

Enciende nftables ahora mismo. Si el servicio no estaba corriendo, lo pone a funcionar.

### Hacer que arranque solo al reiniciar el sistema

```bash
sudo systemctl enable nftables
```

Con esto nftables se enciende automaticamente cada vez que el sistema arranca.
Sin este comando, habria que iniciarlo a mano cada vez.

### Ver si el servicio esta corriendo

```bash
sudo systemctl status nftables
```

Muestra el estado actual del servicio. Si dice "active (running)" esta funcionando.
Si dice "inactive" hay que iniciarlo.

### Reiniciar el servicio

```bash
sudo systemctl restart nftables
```

Apaga y vuelve a encender el servicio. Se usa cuando se hacen cambios en la
configuracion y se quiere que se apliquen de nuevo.
