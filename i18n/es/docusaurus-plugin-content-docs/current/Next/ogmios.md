---
sidebar_label: "Ogmios"
sidebar_position: 7
---

# Ogmios

Del desarrollador en [ogmios.dev/faq](https://ogmios.dev/faq) :
*"Ogmios es una interfaz de puente liviana para `cardano-node`.
Ofrece una API WebSockets que permite a los clientes locales utilizar los mini-protocolos de Ouroboros a través de JSON/RPC.
Ogmios es una solución rápida y liviana que puede implementarse junto con relays para crear puntos de entrada
en la red Cardano para varios tipos de aplicaciones"*.

Es un componente muy conveniente del ecosistema de Cardano que muchos proyectos aprovechan como parte de sus soluciones.
Hoy vamos a configurarlo junto con nuestro `cardano-node` en ejecución.

Primero, obtengamos el binario estático más reciente de la compilación ARM desde el desarrollador.
Este está alojado localmente en nuestro servidor para facilitar el acceso.

```bash
wget http://10.20.20.101:3001/ogmios -O /home/n/preview/bin/ogmios
```

Hagamos que el binario sea ejecutable.

```bash
chmod +x ~/preview/bin/ogmios
```

Hagamos que este endpoint de la API esté disponible externamente.

```bash
sudo ufw allow 1337/tcp
```

Recarga ufw.

```bash
sudo ufw reload
```

Verifica el estado de ufw.

```bash
sudo ufw status
```

Deberías ver el puerto 1337 en la lista.

## Crear un Script

Ahora necesitamos crear un script para iniciar `ogmios`.

```bash
nano ~/preview/scripts/ogmios_start.sh
```

A continuación, configuremos las opciones de inicio correctas.
Notarás que algunas opciones de inicio en `ogmios` son similares a las de `cardano-node`.
Configuraremos el puerto en 1337 y la IP para escuchar en `0.0.0.0`.

Agrega lo siguiente al archivo `ogmios_start.sh` que acabas de abrir en nano.

```bash
#!/bin/bash
#
/home/n/preview/bin/ogmios --node-socket /home/n/preview/socket/node.socket \
--node-config /home/n/preview/config/config.json \
--host 0.0.0.0 \
--port 1337
```

Guarda y sal con `ctrl + o` y `ctrl + x`.

Haz que el script sea ejecutable.

```bash
chmod +x ~/preview/scripts/ogmios_start.sh
```

Probemos el script en nuestra ventana de terminal.

```bash
cd ~/preview/scripts

./ogmios_start.sh
```

Mientras esto se ejecuta, abre una ventana de navegador en tu máquina e ingresa la URL de tu servidor en el puerto `1337`.

Ejemplo: [http://10.20.20.101:1337](http://10.20.20.101:1337)

Deberías ser recibido con un bonito dashboard.

![ogmios](/img/ogmiosdash.png)

Adelante, detén el proceso en ejecución en tu terminal con `ctrl + c`.

A continuación, automatizaremos el script con systemd.

## Crear el Servicio

Crea un archivo de servicio de ejemplo.

```bash
nano ~/preview/scripts/ogmios.service
```

Agrega la siguiente configuración básica de systemd al archivo que acabas de abrir con nano.

```bash
[Unit]
Description       = Ogmios
Wants             = network-online.target
After             = network-online.target

[Service]
User              = n
Type              = simple
WorkingDirectory  = /home/n/preview/scripts/
ExecStart         = /bin/bash -c '/home/n/preview/scripts/ogmios_start.sh'
KillSignal        = SIGINT
RestartKillSignal = SIGINT
LimitNOFILE       = 24000
Restart           = always
RestartSec        = 7
SyslogIdentifier  = ogmios

[Install]
WantedBy          = multi-user.target
```

Copia el archivo de servicio al directorio del sistema.

```bash
sudo cp /home/n/preview/scripts/ogmios.service /etc/systemd/system/
```

Dale al servicio los permisos correctos.

```bash
sudo chmod 0644 /etc/systemd/system/ogmios.service
```

Recarga los daemons.

```bash
sudo systemctl daemon-reload
```

Inicia el servicio `ogmios`.

```bash
sudo systemctl start ogmios.service
```

Verifica el servicio.

```bash
sudo systemctl status ogmios.service
```

Si muestra `active`, podemos habilitar el servicio.

```bash
sudo systemctl enable ogmios.service
```

Ve y abre tu dashboard de `ogmios` nuevamente en tu navegador y observa con orgullo los frutos de tu trabajo.
