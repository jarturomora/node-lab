---
sidebar_label: "Running the Node"
sidebar_position: 2
---

## Ejecutando el Nodo de Cardano

### Adquirir el Nodo

Existen varias opciones para adquirir los binarios `cardano-node` y `cardano-cli`.

- Intersect ofrece binarios estáticos precompilados en su [página de lanzamientos](https://github.com/IntersectMBO/cardano-node/releases)
- También se pueden compilar binarios estáticos o dinámicos desde el [código fuente](https://github.com/IntersectMBO/cardano-node)

Sin embargo, ninguna de estas opciones funcionará para este taller, ya que las Raspberry Pi funcionan con la arquitectura ARM (aarch64).

Aunque las computadoras de placa única Raspberry Pi5 son potentes para su tamaño reducido,
compilar `cardano-node` y `cardano-cli`
desde el código fuente probablemente tomaría toda la duración del taller.

Además, los binarios compilados dinámicamente requieren bibliotecas específicas
(en nuestro caso: libsodium, secp256k1 y blst, cada una de las cuales necesita ser compilada por separado).

Por lo tanto, para este taller nos apoyaremos en los esfuerzos generosos de la
[Armada Alliance](https://armada-alliance.com/),
específicamente en los esfuerzos del pool ZW3RK,
que proporciona binarios compilados estáticamente para aarch64,
lo que significa que estos deberían ejecutarse en la mayoría de las distribuciones de Linux
ya que las bibliotecas dependientes son parte del binario compilado.

Vamos a obtener los binarios estáticos de `cardano-node` y `cardano-cli`
de un servidor local (también una Raspberry Pi5)
y copiarlos al directorio que añadimos a nuestro path `/home/n/preview/bin/`

```bash
wget http://10.20.20.101:3001/cardano-node -O /home/n/preview/bin/cardano-node
wget http://10.20.20.101:3001/cardano-cli -O /home/n/preview/bin/cardano-cli
```

Necesitamos asegurarnos de que los binarios sean ejecutables

```bash
chmod +x /home/n/preview/bin/cardano-node /home/n/preview/bin/cardano-cli
```

Por último, verifiquemos las versiones

```bash
cardano-cli --version
```

Deberías ver la siguiente salida

![cli](/img/cli1.png)

```bash
cardano-node --version
```

Salida nuevamente

![node1](/img/cnodev1.png)

### Ejecutando el Nodo

Para ejecutarlo, el `cardano-node` requerirá algunos archivos de configuración junto con algunos parámetros de inicio.

El nodo requiere los siguientes archivos para funcionar como un nodo básico o relay (no productor de bloques):

- **Archivo de configuración principal**: contiene configuraciones del nodo y apunta a los archivos Genesis de **Shelley**, **Byron**, **Alonzo** y **Conway**.
- **Genesis de Byron**: contiene parámetros iniciales del protocolo e instruye a `cardano-node` cómo arrancar la Era Byron de Cardano.
- **Genesis de Shelley**: contiene parámetros iniciales del protocolo e instruye a `cardano-node` cómo arrancar la Era Shelley de Cardano.
- **Genesis de Alonzo**: contiene parámetros iniciales del protocolo e instruye a `cardano-node` cómo arrancar la Era Alonzo de Cardano.
- **Genesis de Conway**: contiene parámetros iniciales del protocolo e instruye a `cardano-node` cómo arrancar la Era Conway de Cardano.
- **Archivo de topología**: contiene la lista de peers bootstrap, locales y públicos. (Peers son otros nodos ejecutando Cardano)

Vamos a obtener estos archivos

```bash
cd ~/preview/config

wget https://book.world.dev.cardano.org/environments/preview/config.json
wget https://book.world.dev.cardano.org/environments/preview/topology.json
wget https://book.world.dev.cardano.org/environments/preview/byron-genesis.json
wget https://book.world.dev.cardano.org/environments/preview/shelley-genesis.json
wget https://book.world.dev.cardano.org/environments/preview/alonzo-genesis.json
wget https://book.world.dev.cardano.org/environments/preview/conway-genesis.json
```

Ahora que tenemos los archivos necesarios, intentemos ejecutar el nodo.

```bash
cardano-node run --topology ~/preview/config/topology.json \
--database-path ~/preview/test-db \
--socket-path ~/preview/socket/node.socket \
--port 1694 \
--config ~/preview/config/config.json
```

Deberías ver la salida de tu nodo iniciándose en tu ventana de terminal.

![nodestartup](/img/nodestartuptest1.png)

Lo siguiente que me gustaría que hagas es abrir una sesión adicional de SSH a tu servidor Raspberry Pi mientras el nodo se ejecuta.

Una vez conectado, consulta la punta de la cadena para ver qué tan rápido se está sincronizando la blockchain desde cero.

```bash
cardano-cli query tip --testnet-magic 2
```

:::note

Puedes observar la sincronización activa usando el comando watch

```bash
watch -n 1 cardano-cli query tip --testnet-magic 2
```

:::

:::tip

La bandera `--testnet-magic` nos permite especificar las diferentes testnets.
Por ejemplo, preprod sería `--testnet-magic 1`, mientras que mainnet es `--mainnet`.

:::

A estas alturas, debería ser evidente que simplemente no tenemos tiempo para sincronizar desde cero.
Probablemente tomaría toda la duración de la sesión o más para finalizar.

Presiona `ctrl + c` en la sesión donde tienes el nodo ejecutándose actualmente para detener el nodo.

Una vez que el nodo haya sido detenido, elimina la base de datos

```bash
rm -r ~/preview/test-db
```

Si tan solo hubiera una manera más rápida...
