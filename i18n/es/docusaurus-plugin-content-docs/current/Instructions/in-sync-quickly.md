---
sidebar_label: "Catching Up"
sidebar_position: 3
---

# Mithril

Mithril es un protocolo de multi-firma basado en stake para eficiencia y escalabilidad.
Permite la agregación segura de firmas criptográficas
(En este caso, los operadores de stake pools de Cardano que ejecutan firmantes de mithril con acuerdo en un agregador).
Para nuestros propósitos, esto significa que un grupo de SPOs ejecutando firmantes de mithril firman y verifican snapshots regulares.
Los snapshots de la base de datos en sí no están alojados en la cadena (eso sería terrible),
sino que están alojados en un proveedor de nube rápida (Google en este caso).
La seguridad y el propósito provienen de las firmas criptográficas que verifican los snapshots.

Cuanto más stake esté involucrado en firmar snapshots, más seguro será Mithril.

Para nuestros propósitos, descargaremos estos snapshots firmados usando un `mithril-client-cli`.
Sin embargo, esta vez tendremos que compilar nuestros propios binarios.

Afortunadamente para nosotros, este es un proceso relativamente rápido (especialmente en comparación con compilar el robusto `cardano-node` en Haskell).

Como Mithril está construido con Rust, necesitamos instalar el conjunto de herramientas de Rust.

``` sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Presiona Enter en las opciones predeterminadas.

Una vez finalizado, fuentea el directorio env de cargo.

``` sh
. "$HOME/.cargo/env"
```

A continuación, necesitamos instalar algunas dependencias.

``` sh
sudo apt-get install -y libssl-dev make build-essential m4
```

Una vez instaladas, vamos a crear un directorio, clonar el repositorio,
y verificar la versión apropiada para `mithril-client-cli` específicamente.

``` sh
mkdir ~/mithril/
cd ~/mithril/

git clone https://github.com/input-output-hk/mithril.git

cd ~/mithril/mithril/mithril-client-cli

git fetch --tags --all

git checkout 2445.0
```

A continuación, es momento de construirlo.

``` bash
make build
```

:::note

Mientras se construye, esta es una buena oportunidad para levantarse, caminar,
estirar las piernas, tomar un café, hablar sobre política, etc.

:::

:::tip
Compilar puede ser bastante intensivo para el sistema.
Si lo deseas, mientras el `mithril-client` se compila,
abre otra sesión SSH con tu servidor Raspberry Pi y
ejecuta el comando `htop` para ver qué tan duro está trabajando esa pequeña máquina.
![workwork](/img/workingharthtop.png)
:::

Una vez que se haya completado la construcción, copia los archivos al directorio dentro de tu ruta.

``` bash
cp ~/mithril/mithril/mithril-client-cli/mithril-client ~/preview/bin/
```

Verifica tu versión de `mithril-client`.

``` bash
mithril-client --version
```

Tu salida debería verse algo así:

![clientv](/img/mclient.png)

Ahora estamos listos para descargar un snapshot y sincronizar nuestro nodo con la cadena.

Primero, configuremos un par de variables específicas para nuestro agregador y red.
Como realmente solo necesitamos hacer esto una vez,
podemos simplemente crear estas variables durante esta sesión de terminal/ssh.

La primera variable es la red.

``` bash
export CARDANO_NETWORK=preview
```

La segunda es el endpoint del agregador.

``` bash
export AGGREGATOR_ENDPOINT=https://aggregator.pre-release-preview.api.mithril.network/aggregator
```

A continuación, la clave de verificación Genesis.

``` bash
export GENESIS_VERIFICATION_KEY=$(wget -q -O - https://raw.githubusercontent.com/input-output-hk/mithril/main/mithril-infra/configuration/pre-release-preview/genesis.vkey)
```

Por último, el digest del snapshot.

``` bash
export SNAPSHOT_DIGEST=latest
```

Con eso hecho, veamos qué snapshots están disponibles.

``` bash
mithril-client cardano-db snapshot list
```

Como puedes ver, tenemos un menú de snapshots disponibles para nosotros.
(El tuyo se verá un poco diferente ya que la cadena refleja el paso del tiempo
y esta imagen no fue capturada mientras asistes a este taller).

![snapshotlist](/img/snapshotlist1.png)

Ampliemos los detalles de uno de los snapshots.
El snapshot que voy a usar en estos ejemplos no será el más reciente disponible en el momento de este evento,
así que si deseas el snapshot más actualizado,
recomiendo ver tu lista y tomar el más reciente. (Podrías usar el mío,
solo tomará un poco más de tiempo sincronizar, aunque no demasiado).

``` bash
mithril-client cardano-db snapshot show c7694c1bf40ab45022c0c8e5e24f9b0dfeb9410a43e51858d58b2766678bdf75
```

Este comando produce el digest para el snapshot especificado. Esto nos da información crítica como la versión del nodo.

![digest](/img/snapshotdigest15111.png)

:::note

Asegúrate de estar en el directorio de trabajo deseado ya que los snapshots son considerables en tamaño.

:::

Cambia al directorio preview.

``` bash
cd ~/preview
```

Es momento de descargar el snapshot. Toma nota de tu hash digest si estás usando uno más reciente que el de esta guía.

``` bash
mithril-client cardano-db download c7694c1bf40ab45022c0c8e5e24f9b0dfeb9410a43e51858d58b2766678bdf75
```

Deberías ver algo como esto:

![snapdl](/img/downloadsnap1.png)

:::note

Al igual que cuando construimos el `mithril-client`, esto tomará unos minutos.

:::

Una vez que el `mithril-client` haya terminado de descargar el snapshot de la red Preview,
es momento de ejecutar el nodo nuevamente y ver si podemos sincronizarnos.

``` bash
cardano-node run --topology ~/preview/config/topology.json \
--database-path ~/preview/db \
--socket-path ~/preview/socket/node.socket \
--port 1694 \
--config ~/preview/config/config.json
```

Deberías ver nuevamente la salida del inicio del nodo en tu sesión actual.

En otra sesión con tu servidor, revisemos el estado de sincronización.

``` bash
watch -n 1 cardano-cli query tip --testnet-magic 2
```

Podría tomar un par de minutos para que el nodo comience y se sincronice, pero eventualmente deberías ver lo siguiente.

:::info

Si obtienes el error "socket 11 not found",
es porque el `cardano-cli` no puede encontrar el socket para comunicarse con el nodo.
En este caso, si las cosas se hicieron correctamente,
el archivo socket aún no se ha creado como parte del proceso de inicio del nodo.

:::

![syncprog](/img/querytipinsync1.png)

Bien, es hora de detener el nodo nuevamente con `ctrl + c`, pero no te preocupes.
La próxima vez que iniciemos el nodo continuará donde lo dejó.
