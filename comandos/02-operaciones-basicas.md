# Ver, borrar y editar reglas

## Ver todas las reglas activas

```bash
sudo nft list ruleset
```

Muestra todo lo que hay configurado en nftables en este momento.
Tablas, cadenas y reglas. Si no hay nada configurado, no muestra nada.

## Ver las reglas con su numero (handle)

```bash
sudo nft --handle list ruleset
```

Es igual al anterior pero cada regla aparece con un numero llamado handle.
Ese numero se necesita para poder borrar una regla en especifico.

Por ejemplo, la salida puede verse asi:

```
table inet filter {
    chain input {
        tcp dport 22 accept # handle 5
        tcp dport 80 accept # handle 6
    }
}
```

---

## Borrar todas las reglas

```bash
sudo nft flush ruleset
```

Borra absolutamente todas las tablas, cadenas y reglas de un golpe.
Deja nftables en blanco. Se usa para empezar de cero o cuando se quiere
aplicar una configuracion nueva desde un archivo.

## Borrar una regla en especifico

```bash
sudo nft delete rule inet filter input handle 18
```

Borra solo la regla que tiene el numero 18 en la cadena input de la tabla inet filter.
Hay que usar primero `nft --handle list ruleset` para ver cual es el numero de la regla
que se quiere borrar, y luego poner ese numero aqui.

---

## Editar el archivo de configuracion

```bash
sudo nano /etc/nftables.conf
```

Abre el archivo principal de configuracion en el editor nano.
Ahi se pueden escribir las reglas de forma permanente.
Al guardar, los cambios quedan en el archivo pero no se aplican solos,
hay que aplicarlos con el siguiente comando.

## Aplicar las reglas desde un archivo

```bash
sudo nft -f /etc/nftables.conf
```

Lee el archivo que se le indica y aplica todas las reglas que tiene.
Si el archivo tiene errores, nftables lo dice y no aplica nada.
