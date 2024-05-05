# ReLeaf
ReLeaf Wallet: Batch transactions, smart savings, and easy card payments.

<img src="https://i.ibb.co/18R68Ym/New-Project.png">

## Fast Links:

WALLET CODE: [CODE](./ReLeaf/)

PLAYSTORE: [LINK](https://play.google.com/apps/testing/com.altaga.releaf)

# System Diagrams:

Nuestro proyecto tiene 4 componentes fundamentales y sus diagramas son los siguientes.

## Account Abstraction Creation:

<img src="https://i.ibb.co/Gdkjxym/Re-Leaf-Diagram-Account-Abstraction-drawio.png">

En nuestro modelo de Account Abstraction, todas las cuentas que tengamos aparte de la cuenta principal, sera una [Light Account](https://github.com/alchemyplatform/light-account), la cual es una implementacion de ERC-4337 propuesta por Alchemy e implementada en Moonbeam por nuestro equipo, el codigo de nuestra implementacion podras encontrar todos los archivos en la pestaña de [Contracts](./Contracts/).

- [Entrypoint](./Contracts/core/EntryPoint.sol): Este contrato sirve como punto de entrada para ejecutar comandos en las AA wallets.
    - EntryPoint Deployment Address: https://moonscan.io/address/0x0835980A1f2f32A12CA510E73bE3954D9F437114    
- [ReLeafAccountFactory](./Contracts/ReLeafAccountFactory.sol): Este contrato facilita la creacion de los AA wallets para poder mejorar la UI/UX del usuario al crear su wallet.
    - ReLeaf Account Factory: https://moonscan.io/address/0x2D991fE2767FF819F6b3dab83625d331ecCe61a3
- [ReLeafAccount](./Contracts/ReLeafAccount.sol): Ese Contrato por ultimo es la interfaz que se desplegara en la chain una vez la creemos con el Factory, todas las operaciones que se realicen sobre esta wallet deberan realizarse desde la cuenta owner.

## Batch Balances:

<img src="https://i.ibb.co/yhgq6zR/Re-Leaf-Diagram-Balances-drawio.png">

Siguiendo un poco los objetivos de los precompiles de Moonbeam, hemos realizado nuestro propio "precompile" contract el cual sirve para poder realizar una llamada en batch de multiples tokens para obtener sus balances y decimales desde una sola RPC Call. [MORE DETAILS](./ReLeaf/src/screens/main/tabs/tab1.js)

- [Batch Token Balances](./Contracts/ReLeafBalances.sol): Este contrato sirve para realizar batch token balances.
    - ReLeafBalances Deployment Address: https://moonscan.io/address/0xcdEE75520dcE5240C39a308A705Ed3D6c6D82664  

## Batch Payments:

<img src="https://i.ibb.co/f2nxn9T/Re-Leaf-Diagram-Payments-drawio.png">

Uno de los componentes mas importantes en nuestro proyecto es el uso del [Batch Precompile](https://docs.moonbeam.network/builders/pallets-precompiles/precompiles/batch/) para mejorar enormemente la UI/UX del usuario, ya que podemos realizar transacciones mas complejas desde una sola firma al RPC.

- Direct Transfer: Podemos realizar una serie de transacciones en batch ya sea de tokens nativos o ERC20, todos en una sola transaccion. [MORE DETAILS](#send)

    - Si esta encendida la cuenta de savings, segun el protocolo de saving que este activado, se adjuntara una transaccion extra al batch el cual mandara dinero a esa cuenta. [MORE DETAILS](#savings) 

- Card Payment: Podemos realizar pagos sin necesidad de nuestra wallet, unicamente desde nuestra tarjeta ligada a nuestra Account Abstraction Card Wallet. [MORE DETAILS](#payment)

## Swap Tokens:

<img src="https://i.ibb.co/0tf7gSV/Re-Leaf-Diagram-Swap-drawio.png">

De la misma forma que en los batch paymens, hemos realizado el codigo necesario para poder mejorar la user experience de el swap de Stellaswap para realizar todos los swaps desde una sola transaccion en batch. [MORE DETAILS](#swap)

- Swap SDK: se tomo como base el swap-sdk de stella swap, pero se adapto para funcionar con la funcion Swap.
    - Swap-SDK: https://github.com/stellaswap/swap-sdk/tree/main

# Screens:

ReLeaf es una solucion integral de pagos, savings y tarjetas. Integrado los SDK y Precompiles del ecosistema de Moonbeam en una sola aplicacion.

## Wallet:

Como base para utilizar todos los servicios y recursos de Moonbeam, creamos una wallet sencilla, la cual nos permite visualizar los Assets en la red de moonbeam. [CODE](./ReLeaf/src/screens/main/tabs/tab1.js)

<img src="https://i.ibb.co/m9gwVzc/vlcsnap-2024-05-05-13h55m07s508.png" width="32%">

A su vez esta tab integra el contrato de [ReLeafBalances](./Contracts/ReLeafBalances.sol), el cual nos permite obtener todos los balances de todos los X-Tokens en el ecosistema de Moonbeam desde una sola RPC Call.

    const tokenBalances = new ethers.Contract(
        BatchTokenBalancesAddress,
        abiBatchTokenBalances,
        this.provider,
    );
    const [balanceTemp, tempBalances, tempDecimals] = await Promise.all([
        this.provider.getBalance(publicKey),
        tokenBalances.batchBalanceOf(publicKey, tokensArray),
        tokenBalances.batchDecimals(tokensArray),
    ]);

## Send:

Con la funcion de send, podemos realizar envios de native tokens o ERC20 tokens en batch, lo que quiere decir que todas las transacciones se ejecutaran en la misma transaccion. [CODE](./ReLeaf/src/screens/sendWallet/sendWallet.js)

<img src="https://i.ibb.co/XV1BYJQ/vlcsnap-2024-05-05-13h55m25s064.png" width="32%"> <img src="https://i.ibb.co/K6ztVJz/vlcsnap-2024-05-05-13h55m28s247.png" width="32%"> <img src="https://i.ibb.co/xJXSbNg/vlcsnap-2024-05-05-13h55m31s607.png" width="32%">

Es muy importante revisar que todas las transacciones son ejecutadas mediante el [Batch Precompile](https://docs.moonbeam.network/builders/pallets-precompiles/precompiles/batch/) Contract de Moonbeam. 

    this.batchContract = new ethers.Contract(
        BatchTransactionsAddress,
        abiBatch,
        this.provider,
    );
    ...
    const transactionBatch =
    await this.batchContract.populateTransaction.batchAll(
        [...transactions.map(item => item.to)],
        [...transactions.map(item => item.value)],
        [...transactions.map(item => item.data)],
        [],
        {
        from: this.state.publicKey,
        },
    );

## Receive:

Con esta pantalla, puedes de forma sencilla mostrar tu Wallet para recibir fondos, ya sea native tokens o ERC20. [CODE](./ReLeaf/src/screens/depositWallet/depositWallet.js) 

<img src="https://i.ibb.co/bbBZ17k/vlcsnap-2024-05-05-13h55m38s880.png" width="32%">

## Swap:

Gracias a Stella Swap y a que su SDK es publico, podemos realizar swaps entre tokens, ya sea nativo o ERC20. [CODE](./ReLeaf/src/screens/swapWallet/swapWallet.js)

<img src="https://i.ibb.co/hLSy59K/vlcsnap-2024-05-05-13h55m45s194.png" width="32%"> <img src="https://i.ibb.co/hVdNSD4/vlcsnap-2024-05-05-13h55m48s943.png" width="32%">

Como se comento antes, a diferencia de un swap desde la plataforma de StellaSwap, nosotros podemos realizar todo el swap mediante una sola transaccion mediante Batch Precompile.

    const batchData = this.batchContract.interface.encodeFunctionData(
        'batchAll',
        [
        [this.state.tokenSelected1.address, AGGREGATOR_ADDRESS],
        [],
        [allowance, data],
        [],
        ],
    );

## Payment:

<img src="https://i.ibb.co/HrcQdHM/vlcsnap-2024-05-05-13h56m06s927.png" width="32%"> <img src="https://i.ibb.co/RY8bLkw/vlcsnap-2024-05-05-13h56m11s820.png" width="32%"> <img src="https://i.ibb.co/HYkCXq9/vlcsnap-2024-05-05-13h56m25s677.png" width="32%">

## Savings:


<img src="https://i.ibb.co/P9nb73m/vlcsnap-2024-05-05-13h56m32s908.png" width="32%"> <img src="https://i.ibb.co/28VTxY7/vlcsnap-2024-05-05-13h56m38s244.png" width="32%"> <img src="https://i.ibb.co/NKp3rgR/vlcsnap-2024-05-05-13h56m47s323.png" width="32%">


## Cards:



<img src="https://i.ibb.co/bdFpD77/vlcsnap-2024-05-05-13h57m37s280.png" width="32%"> <img src="https://i.ibb.co/Sm51S8r/vlcsnap-2024-05-05-13h57m44s761.png" width="32%"> <img src="https://i.ibb.co/QCSfqTK/Screenshot-20240505-150921.png" width="32%">



<img src="https://i.ibb.co/23f62Rb/Screenshot-20240505-150943.png" width="32%"> <img src="https://i.ibb.co/w7XWsVD/Screenshot-20240505-150949.png" width="32%"> <img src="https://i.ibb.co/0rpNSNn/Screenshot-20240505-150955.png" width="32%">
