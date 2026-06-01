# Guardar la configuracion

## El problema con las reglas en vivo

Cuando se agregan reglas con `nft add rule`, esas reglas se aplican de inmediato
pero solo existen en memoria. Si el sistema se reinicia, todas las reglas desaparecen.
Para que se queden hay que guardarlas en el archivo de configuracion.

---

## Guardar las reglas activas en el archivo

```bash
sudo nft list ruleset > /etc/nftables.conf
```

Este comando hace dos cosas juntas:
- `nft list ruleset` lee todas las reglas que estan activas ahora mismo
- `> /etc/nftables.conf` guarda ese resultado en el archivo de configuracion

Despues de esto, si el sistema reinicia y nftables esta habilitado,
va a leer ese archivo y cargar las reglas automaticamente.

---

## Aplicar las reglas desde el archivo

```bash
sudo nft -f /etc/nftables.conf
```

Lee el archivo y aplica todo lo que tiene.
Se usa cuando se edita el archivo a mano o cuando se quiere recargar la configuracion.

---

## Flujo completo recomendado

```bash
# Paso 1 - Crear o modificar reglas en vivo y probar que funcionan
sudo nft add rule inet filter input tcp dport 8080 accept

# Paso 2 - Verificar que las reglas estan como deben estar
sudo nft list ruleset

# Paso 3 - Guardar todo en el archivo para que persista
sudo nft list ruleset > /etc/nftables.conf

# Paso 4 - Habilitar el servicio para que cargue las reglas al iniciar
sudo systemctl enable nftables
```

El orden importa: primero probar que funciona, despues guardar.
Si se guarda algo que esta mal configurado, al reiniciar el sistema va a cargar
las reglas malas automaticamente.
