---
sidebar_label: "Getting Started"
sidebar_position: 1
---

# Introducción

### Bienvenido al taller de nodos de Cardano

## Conexión a tu servidor vía SSH

Lo primero que necesitamos hacer es conectarnos a tu servidor mediante secure shell (SSH).

Mac OSX y Linux (la mayoría de las distribuciones) ya tienen el cliente SSH instalado. Si tienes una máquina Windows, necesitarás instalar [putty](https://www.putty.org/).

Se te entregará una tarjeta con la IP de tu servidor, usuario y contraseña impresos en ella.
Usarás estas credenciales para iniciar sesión en tu servidor Raspberry Pi mediante SSH.

:::warning

Hoy no usaremos las mejores prácticas para SSH, ya que estamos relajando la seguridad para facilitar este ejercicio práctico.
En un entorno de producción necesitarás reforzar tus métodos de conexión SSH al servidor.

Los siguientes son ejemplos de cómo reforzar SSH:

- Cambiar el puerto SSH predeterminado del 22 a un número mucho más alto.
  El puerto 22 es comúnmente escaneado y atacado mediante ataques de fuerza bruta y los escáneres suelen comenzar desde números bajos al buscar puertos abiertos.
- Deshabilitar la autenticación por contraseña y usar solo claves SSH generadas.
- Configurar el firewall para permitir conexiones SSH solo desde IPs específicas que poseas.
- Instalar `fail2ban` o algo similar para limitar los intentos de autenticación en tu servidor.
- Configurar un túnel entre dispositivos usando algo como `wireguard` o `openvpn` entre cliente y servidor.

:::

Primero, necesitamos conectarnos a tu dispositivo con SSH.

Para Mac OSX y Linux, usa el siguiente comando:

``` bash
ssh n@10.20.20.X
```

Para los usuarios de putty, asegúrate de que el puerto sea `22`, el protocolo o tipo de conexión sea `ssh` y que tu IP sea correcta. Luego `abre` la conexión.

![putty1](/img/putty1.png)

:::tip

Reemplaza la `X`, o lo que sea que esté en el último octeto en `10.20.20.X`, con la información correcta en tu tarjeta de credenciales.

:::

## Es hora de comenzar

Una vez que te hayas conectado, ¡es hora de comenzar!

Antes de avanzar, verifiquemos si hay actualizaciones/mejoras en nuestro servidor.

``` bash
sudo apt-get update -y && sudo apt-get upgrade -y
```

A continuación, creemos algunos directorios de trabajo:

``` bash
cd
mkdir -p preview/{node,config,socket,test-db,scripts,bin,logs}
```

Ahora, añadamos algunos elementos de ruta a nuestro archivo `.bashrc`.

Abre tu archivo `.bashrc` con un editor de texto:

``` bash
nano ~/.bashrc
```

Luego, copia las siguientes líneas de exportación y pégalas al final del archivo `.bashrc`:

``` bash
export PATH="/home/n/preview/bin:$PATH"
export CARDANO_NODE_SOCKET_PATH="/home/n/preview/socket/node.socket"
```

Guarda con `ctrl + o` y sal del editor con `ctrl + x`.

Fuenteemos nuestro archivo bash y luego verifiquemos nuestras rutas:

``` bash
cd
. .bashrc
echo $PATH $CARDANO_NODE_SOCKET_PATH
```

Deberías ver las rutas `/home/n/preview/bin:/home/n/` y `/home/n/preview/socket/node.socket` como parte de la salida.
