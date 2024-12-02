---
sidebar_label: "Keys and Addresses"
sidebar_position: 6
---

# Claves y Direcciones

## Claves

Ahora vamos a crear claves de pago, claves de staking, direcciones, y también enviar una transacción simple.

### Claves de Pago

Comencemos creando un par de directorios para alojar nuestras claves y direcciones.

:::danger

No estamos siguiendo las mejores prácticas aquí.
Si estás creando pares de claves de pago u otras claves que requieren seguridad en mainnet o que interactúan con fondos reales,
**DEBES** almacenar las claves secretas de manera offline.

:::

``` bash
mkdir -p  ~/preview/{wallet1,wallet2}
cd ~/preview/wallet1
```

El primer par de claves que vamos a crear es el par de claves de pago para nuestro "wallet1".
Este conjunto de claves permite mantener y transferir ada y otros activos nativos.

``` bash
cardano-cli address key-gen \
--verification-key-file ~/preview/wallet1/payment.vkey \
--signing-key-file ~/preview/wallet1/payment.skey
```

### Claves de Staking

El siguiente par que vamos a crear es un par de claves de staking para wallet1.
Esto nos permite generar una dirección usando **AMBAS** claves de pago y staking que nos permite hacer staking en la blockchain de Cardano.
Podemos derivar una dirección usando **solo** las claves de pago,
y eso se consideraría una dirección _Enterprise_, que no puede participar en el staking.

``` bash
cardano-cli conway stake-address key-gen \
--verification-key-file ~/preview/wallet1/stake.vkey \
--signing-key-file ~/preview/wallet1/stake.skey
```

### Dirección

A continuación, vamos a construir una dirección usando las claves de verificación (públicas) del par de claves de pago y staking que acabamos de crear para wallet1.

``` bash
cardano-cli address build \
--payment-verification-key-file ~/preview/wallet1/payment.vkey \
--stake-verification-key-file stake.vkey \
--testnet-magic 2 \
--out-file ~/preview/wallet1/payment.addr
```

Una vez hecho esto, concatena el archivo payment.addr.

``` bash
cat ~/preview/wallet1/payment.addr
```

![paymentaddr](/img/paymentaddrw1.png)

Copia esta salida y envíame tu dirección de pago por correo electrónico a jesse.smith@iohk.io

:::info

Normalmente, una persona usaría el faucet de la testnet Preview para solicitar fondos,
pero el faucet tiene medidas incorporadas para evitar que bots o individuos realicen solicitudes de fondos repetidamente.

:::

### Wallet 2

Mientras trabajamos en enviar fondos a los participantes, sigamos al siguiente paso y creemos nuestra segunda "wallet", wallet2.

``` bash
cd ~/preview/wallet2
```

Esta vez, vamos a crear una dirección Enterprise simple sin la capacidad de participar en el staking.

Primero, el par de claves de pago nuevamente.

``` bash
cardano-cli address key-gen \
--verification-key-file ~/preview/wallet2/payment.vkey \
--signing-key-file ~/preview/wallet2/payment.skey
```

### Dirección Enterprise

A partir de aquí, simplemente construimos la dirección usando nuestro archivo de clave de verificación.

``` bash
cardano-cli address build \
--payment-verification-key-file ~/preview/wallet2/payment.vkey \
--out-file ~/preview/wallet2/payment.addr \
--testnet-magic 2
```

Una vez que haya enviado ada a la primera dirección que creaste (en wallet1), consultemos los UTxOs pertenecientes a tu dirección de pago.

``` bash
cardano-cli query utxo --address $(cat ~/preview/wallet1/payment.addr) --testnet-magic 2
```

:::tip

Esto puede tomar un segundo, no entres en pánico.

:::

Deberías ver una salida que contiene el TxHash, el valor de TxIx y la cantidad de ada contenida en el UTxO en lovelaces.

![utxo1](/img/utxo1.png)

:::note

1000000 lovelaces equivalen a 1 ada. Al usar `cardano-cli`, siempre ingresaremos valores en lovelaces.

:::

Una vez que confirmes que tu wallet tiene un UTxO válido con una cantidad de ada asociada,
crearemos y enviaremos una transacción simple usando `cardano-cli`.

Primero, hagamos un directorio para alojar nuestros archivos de transacción.

``` bash
mkdir ~/preview/tx
cd ~/preview/tx
```

Para esta primera transacción, enviaremos una pequeña cantidad de ada de "wallet1" a "wallet2".

Construyamos la transacción.

:::tip

Please ensure you replace the carroted utxo hash text and the txix text with your actual utxo
information obtained form the `query utxo` command we did earlier.

:::

``` bash
cardano-cli conway transaction build \
--socket-path /home/n/preview/socket/node.socket \
--testnet-magic 2 \
--tx-in <the utxo hash you want to consume>#<the txix of the utxo> \
--change-address $(cat /home/n/preview/wallet1/payment.addr) \
--tx-out $(cat /home/n/preview/wallet2/payment.addr)+10000000 \
--out-file ~/preview/tx/tx1.raw
```

Deberías ver en la parte inferior del comando una tarifa estimada en lovelace para la transacción.

![fee1](/img/estfee1.png)

Una vez que hayamos construido el archivo de transacción (`tx1.raw`), firmemos la transacción con el `payment.skey` de wallet1.

``` bash
cardano-cli conway transaction sign --tx-body-file ~/preview/tx/tx1.raw \
--signing-key-file ~/preview/wallet1/payment.skey \
--out-file ~/preview/tx/tx1.signed
```

Una vez que hayamos firmado la transacción, ¡es hora de enviarla!

``` bash
cardano-cli conway transaction submit --tx-file ~/preview/tx/tx1.signed \
--socket-path /home/n/preview/socket/node.socket \
--testnet-magic 2
```

Si todo se hizo correctamente, serás recibido con una notificación de éxito.

![success](/img/txsub1.png)

Ahora verifiquemos las direcciones de wallet1 y wallet2 para ver qué UTxOs contienen ahora.

``` bash
cardano-cli query utxo --address $(cat ~/preview/wallet1/payment.addr) --testnet-magic 2
```

![utx1](/img/w1utxo1.png)

y para wallet2:

``` bash
cardano-cli query utxo --address $(cat ~/preview/wallet2/payment.addr) --testnet-magic 2
```

![utx2](/img/w2utxo1.png)

¿Qué notas?

:::info

En este punto, hemos creado una transacción con `cardano-cli` de la manera más simple posible. Sin embargo, podemos añadir más a lo que acabamos de hacer.

Ejercicios de crédito extra:

- Crea una wallet3.
- Envía 10 ada a wallet3 y wallet2 desde wallet1, consumiendo un único UTxO.
- Envía 10 ada desde wallet1 y wallet2 a wallet3 (20 ada en total) en una sola transacción.
- Crea una transacción usando `build-raw` en lugar de `build` (necesitarás calcular manualmente la tarifa, lovelaces, cambio, ttl, etc.).

Si deseas intentar construir una transacción con `build-raw`, sigue las instrucciones aquí:
[developers.cardano.org](https://developers.cardano.org/docs/get-started/cardano-cli/get-started/simple-transactions)
