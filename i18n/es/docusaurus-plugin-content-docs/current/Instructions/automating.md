---
sidebar_label: "Automating Things"
sidebar_position: 4
---

# Automatizando el Nodo

:::info

Antes de avanzar demasiado, me gustaría hablar sobre las opciones RTS para `cardano-node`.

:::

**RTS** significa Runtime System, que es una capa de software para programas Haskell que permite la personalización de cosas como:

- Gestión de Memoria
- Recolección de Basura
- Concurrencia y paralelismo
- Manejo de excepciones

Las opciones RTS son una característica específica de Haskell que el `cardano-node` aprovecha específicamente.

Estas opciones pueden ser implementadas:

- durante la compilación del nodo
- como un flag de anulación mientras se ejecuta el nodo

Podemos verificar las opciones RTS integradas en nuestro binario estático `cardano-node`:

``` bash
cardano-node +RTS --info
```

Cuando construyamos nuestro script, anularemos una opción RTS para aumentar los threads que `cardano-node` consume de 2 a 4.

## Construyendo el script

Lo primero que necesitamos para automatizar la ejecución de nuestro nodo es crear un script de inicio.

``` bash
nano ~/preview/scripts/node.sh
```

Añade lo siguiente al script `node.sh` que acabamos de crear:

``` bash
#!/bin/bash
#
# Configuraremos algunas variables para acortar nuestro comando de inicio
# Ruta de la base de datos
DB=/home/n/preview/db
# Ruta del socket
SOCKET=/home/n/preview/socket/node.socket
# Archivo de configuración
CONFIG=/home/n/preview/config/config.json
# Archivo de topología
TOPOLOGY=/home/n/preview/config/topology.json
# Dirección del host. Lo configuraremos para 'escuchar' conexiones entrantes
HOST=0.0.0.0
# Cambia <your port> al puerto de tu nodo productor de bloques o relay
PORT=1694
#
# Comando para ejecutar el nodo
#
/home/n/preview/bin/cardano-node run --topology $TOPOLOGY \
    --database-path $DB \
    --socket-path $SOCKET \
    --port $PORT \
    --config $CONFIG \
    --host-addr $HOST +RTS -N4
```

Una vez añadido, guarda con `ctrl + o` y sal con `ctrl + x`.

Notarás que las variables configuradas en este script bash coinciden más o menos con lo que ingresamos al iniciar el nodo en la línea de comandos,
con la excepción de la opción RTS añadida al final del comando para ejecutar el nodo.

Haz que el script sea ejecutable.

``` bash
chmod +x ~/preview/scripts/node.sh
```

Prueba tu script:

``` bash
cd ~/preview/scripts

./node.sh
```

Deberías ver una salida similar a la siguiente:

![scrptop](/img/testnodescript.png)

Detén el proceso nuevamente con `ctrl + c`.

## Creando el Servicio Systemd

Vamos a crear el archivo de servicio en el directorio de scripts que creamos anteriormente:

``` bash
cd ~/preview/scripts

nano node.service
```

Añade lo siguiente a nuestro archivo de servicio.
Este archivo es muy básico; dependiendo de tus necesidades, puedes querer añadir, cambiar o hacer cosas de manera diferente.

``` bash
[Unit]
Description       = Cardano Node
Wants             = network-online.target
After             = network-online.target

[Service]
User              = n
Type              = simple
WorkingDirectory  = /home/n/preview
ExecStart         = /bin/bash -c '/home/n/preview/scripts/node.sh'
KillSignal        = SIGINT
RestartKillSignal = SIGINT
LimitNOFILE       = 24000
Restart           = always
RestartSec        = 7
SyslogIdentifier  = cardano-node

[Install]
WantedBy          = multi-user.target
```

Guarda el archivo con `ctrl + o` y sal con `ctrl + x`.

Ahora, copiemos nuestro archivo al directorio donde los servicios están ubicados en nuestro sistema:

``` bash
sudo cp ~/preview/scripts/node.service /etc/systemd/system/
```

También necesitamos asegurarnos de que el archivo de servicio tenga los permisos apropiados:

``` bash
sudo chmod 0644 /etc/systemd/system/node.service
```

A continuación, necesitamos recargar el demonio para que nuestro sistema reconozca el nuevo archivo de servicio:

``` bash
sudo systemctl daemon-reload
```

:::tip

Cada vez que se cambie un archivo de servicio, necesitas recargar el demonio systemd.

:::

Iniciemos nuestro nuevo servicio de nodo:

``` bash
sudo systemctl start node.service
```

Ahora necesitamos asegurarnos de que nuestro servicio se haya iniciado exitosamente:

``` bash
sudo systemctl status node.service
```

Si todo ha salido bien, deberíamos ver una salida que indique que el servicio está `active`:

![active](/img/nodeserviceactive.png)

Si está activo, necesitamos habilitar el servicio para que se ejecute automáticamente al iniciar el sistema.

``` bash
sudo systemctl enable node.service
```

Verás que se ha creado un enlace simbólico para el servicio:

![symlink](/img/enabledsymlink.png)

Una vez hecho esto, systemd se asegurará de que el `cardano-node` esté ejecutándose como especificamos en segundo plano, listo para usar.

Para asegurarnos, consultemos la punta de la cadena:

``` bash
cardano-cli query tip --testnet-magic 2
```

¡Con suerte, estás sincronizado!

![sync](/img/querytipinsync1.png)
