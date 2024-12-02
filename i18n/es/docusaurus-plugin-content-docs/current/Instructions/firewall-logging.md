---
sidebar_label: "Logging, Firewall, and Monitoring"
sidebar_position: 5
---

# Registro, Firewall y Monitoreo

## Un par de Daemons

Antes de avanzar, hablemos e instalemos un par de servicios en nuestro servidor.

El primero es **fail2ban**

**fail2ban** es un daemon que monitorea y bloquea clientes que fallan repetidamente en las verificaciones de autenticación, como intentos de fuerza bruta en SSH.

Es una buena idea ejecutar esto en cualquier servidor accesible públicamente.

``` bash
sudo apt-get install -y fail2ban
```

El otro servicio genérico que me gustaría instalar es **chrony**

**chrony** es una implementación del Protocolo de Tiempo en Red (Network Time Protocol, NTP).
**Cardano** depende mucho del tiempo y requiere que los nodos productores de bloques, los relays y cualquier otro nodo mantengan un tiempo preciso.
Esto es especialmente crítico para los nodos productores de bloques, ya que la red espera que los bloques sean creados y propagados dentro de un segundo (slot).
Con la configuración predeterminada, **chrony** verificará regularmente con servidores NTP públicos.
En nuestro caso, verificará con `pool.ntp.org`.
Aunque podemos configurar muchas opciones en `/etc/chrony.conf`,
como especificar un servidor NTP de menor estrato,
la configuración predeterminada debería ser más que adecuada para nuestros propósitos.

``` bash
sudo apt-get install -y chrony
```

Una vez instalado, verifica que **chrony** mantenga tu servidor sincronizado.

``` bash
chronyc tracking
```

![chrony](/img/chronyct.png)

## Firewall

Ejecutar cualquier infraestructura accesible por internet requiere tomar medidas de seguridad,
y el firewall es una de las más importantes.
Hoy usaremos **ufw** (firewall sencillo) que viene incluido con nuestra distribución Linux del servidor Ubuntu.
**ufw** es simplemente un frontend simple y fácil de usar para **iptables**,
que nos permite configurar reglas de filtrado de paquetes IP del firewall del kernel de Linux.

Primero, configuremos un par de reglas predeterminadas.

La primera es denegar el tráfico entrante.

``` bash
sudo ufw default deny incoming
```

La siguiente es permitir el tráfico saliente.

``` bash
sudo ufw default allow outgoing
```

Asegúrate de permitir que el servicio **ssh** sea accesible en tu servidor.

``` bash
sudo ufw allow ssh
```

:::tip

En producción, es una buena práctica cambiar el puerto que usa tu servidor SSH a algo más alto y menos común (el puerto predeterminado es el 22),
ya que hay muchos escáneres que buscan específicamente el puerto 22,
y muchos escáneres comienzan a escanear desde un número bajo.
Hoy no haremos esto por un par de razones:
tiempo, estos no son accesibles públicamente y estos servidores no son infraestructura crítica.

:::

:::danger

¡Si omites este paso y habilitas el firewall, ya no podrás acceder a tu servidor!

:::

Hagamos que el puerto de nuestro `cardano-node` esté disponible.

``` bash
sudo ufw allow 1694/tcp
```

:::info

El protocolo nodo-a-nodo (NtN) de `cardano-node` fue diseñado para facilitar la comunicación bidireccional sobre una sola conexión de `TCP/IP` saliente.
Esto significa que podemos conectarnos con muchos otros nodos incluso si no permitimos conexiones entrantes en nuestro puerto especificado.

:::

A continuación, necesitamos abrir el puerto para el exportador de nodo de Prometheus (hablaremos más de esto en un momento...).

``` bash
sudo ufw allow 9100/tcp
```

Luego, abramos el puerto para el exportador Prometheus de nuestro `cardano-node`.

``` bash
sudo ufw allow 12798/tcp
```

Finalmente, habilitemos el firewall.

``` bash
sudo ufw enable
```

Luego recárgalo.

``` bash
sudo ufw reload
```

Si todo se hizo correctamente, tu sesión SSH debería seguir activa.

Obtenemos una vista general de nuestras reglas rápidamente.

``` bash
sudo ufw status
```

![ufwstat](/img/ufwstatus.png)

## Registro y Monitoreo

Lo siguiente que necesitamos hacer es habilitar algunos registros y monitoreo para nuestro nodo.

`cardano-node` viene con la capacidad de producir métricas **Prometheus** por defecto.

:::info

**Prometheus** es un sistema de monitoreo de código abierto.

:::

Para permitir que un servidor **Prometheus** obtenga datos de nuestro nodo, necesitamos realizar un ajuste en nuestro archivo de configuración del nodo.

``` bash
nano ~/preview/config/config.json
```

Busca la línea `"hasPrometheus"` y cambia la IP del localhost `127.0.0.1` a `0.0.0.0` para escuchar.

``` bash
  "hasPrometheus": [
    "0.0.0.0",
    12798
  ],
```

Aún no estamos listos para guardar y salir del archivo; mientras estamos aquí editando, hagamos algunos ajustes para habilitar el registro.

Primero cambiaremos la ubicación predeterminada de los registros. Esto especificará dónde se escriben los registros si no se configura un scribe.

Busca `"defaultScribes"` y cámbialo de Stdout a FileSK, con una ruta al directorio de registros que creamos anteriormente.

``` bash
  "defaultScribes": [
    [
      "FileSK",
      "/home/n/preview/logs/cardano.json"
    ]
  ],
```

Ahora especificaremos la salida de `"setupScribes"`. Encuentra la línea `"setupScribes"` y modifícala para que coincida con lo siguiente.

``` bash
    "setupScribes": [
    {
      "scFormat": "ScJson",
      "scKind": "FileSK",
      "scName": "/home/n/preview/logs/cardano.json",
      "scRotation": null
    }
  ]
```

:::tip

Ten cuidado de preservar el formato `.json` al ajustar estos.
Es fácil romper el archivo de configuración y evitar que tu nodo se inicie al eliminar accidentalmente una coma
o causar algún otro error de sintaxis.

:::

Una vez que se haya actualizado tu archivo `config.json`, guárdalo con `ctrl + o` y sal con `ctrl + x`.

Para activar estos cambios, necesitamos reiniciar el nodo.

``` bash
sudo sytemctl restart node.service
```

Dale unos segundos y luego verifica que el servicio esté en ejecución.

``` bash
sudo systemctl status node.service
```

También puedes consultar la punta nuevamente.

``` bash
cardano-cli query tip --testnet-magic 2
```

:::tip

Si recibes un error de "socket not found" al hacer esto, pero tu `node.service` está activo,
dale un minuto al nodo para que se inicie hasta que se cree el archivo socket y vuelve a intentarlo.

:::

Echemos un vistazo en vivo a nuestros registros con el comando tail.

``` bash
tail -f ~/preview/logs/cardano.json
```

Deberías ver la salida en vivo de los registros de tu `cardano-node` escribiéndose en el archivo `cardano.json`.

`ctrl + c` para finalizar el comando.

A continuación, instalemos el exportador de nodo de **Prometheus**.

``` bash
sudo apt-get install -y prometheus-node-exporter
```

Esto inicia automáticamente el servicio y el servidor **Prometheus** que he configurado debería poder obtener datos de tu servidor como un endpoint.
Deberías poder ver las estadísticas de tu nodo aparecer en el panel de **Grafana** ahora.
