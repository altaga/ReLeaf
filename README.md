# ReLeaf
ReLeaf Wallet: Batch transactions, smart savings, and easy card payments.

<img src="https://i.ibb.co/18R68Ym/New-Project.png">

## Fast Links:

WALLET CODE: [CODE](./ReLeaf/)

PLAYSTORE: [LINK](https://play.google.com/apps/testing/com.altaga.releaf)

# System Diagrams:

Our project has 4 fundamental components and their diagrams are as follows.

## Account Abstraction Creation:

<img src="https://i.ibb.co/Gdkjxym/Re-Leaf-Diagram-Account-Abstraction-drawio.png">

In our Account Abstraction model, all the accounts we have apart from the main account will be a [Light Account](https://github.com/alchemyplatform/light-account), which is a proposed ERC-4337 implementation by Alchemy and implemented in Moonbeam by our team, the code of our implementation you can find all the files in the tab [Contracts](./Contracts/).

- [Entrypoint](./Contracts/core/EntryPoint.sol): This contract serves as an entry point to execute commands in the AA wallets.
    - EntryPoint Deployment Address: https://moonscan.io/address/0x0835980A1f2f32A12CA510E73bE3954D9F437114    
- [ReLeafAccountFactory](./Contracts/ReLeafAccountFactory.sol): This contract facilitates the creation of AA wallets in order to improve the user's UI/UX when creating their wallet.
    - ReLeaf Account Factory: https://moonscan.io/address/0x2D991fE2767FF819F6b3dab83625d331ecCe61a3
- [ReLeafAccount](./Contracts/ReLeafAccount.sol): Lastly, this Contract is the interface that will be deployed in the chain once we create it with the Factory, all operations carried out on this wallet must be carried out from the owner account.

## Batch Balances:

<img src="https://i.ibb.co/yhgq6zR/Re-Leaf-Diagram-Balances-drawio.png">

Following a little the objectives of the Moonbeam precompiles, we have made our own "precompile" contract which serves to be able to make a batch call of multiple tokens to obtain their balances and decimals from a single RPC Call. [MORE DETAILS](./ReLeaf/src/screens/main/tabs/tab1.js)

- [Batch Token Balances](./Contracts/ReLeafBalances.sol): This contract is used to carry out batch token balances.
    - ReLeafBalances Deployment Address: https://moonscan.io/address/0xcdEE75520dcE5240C39a308A705Ed3D6c6D82664  

## Batch Payments:

<img src="https://i.ibb.co/f2nxn9T/Re-Leaf-Diagram-Payments-drawio.png">

One of the most important components in our project is the use of [Batch Precompile](https://docs.moonbeam.network/builders/pallets-precompiles/precompiles/batch/) to greatly improve the user's UI/UX, since we can perform more complex transactions from a single signature to the RPC.

- Direct Transfer: We can carry out a series of batch transactions with either native or ERC20 tokens, all in a single transaction. [MORE DETAILS](#send)

     - If the savings account is turned on, according to the saving protocol that is activated, an extra transaction will be attached to the batch which will send money to that account. [MORE DETAILS](#savings)

- Card Payment: We can make payments without needing our wallet, only from our card linked to our Account Abstraction Card Wallet. [MORE DETAILS](#payment)

## Swap Tokens:

<img src="https://i.ibb.co/0tf7gSV/Re-Leaf-Diagram-Swap-drawio.png">

In the same way as in batch paymens, we have made the necessary code to improve the user experience of the Stellaswap swap to carry out all swaps from a single batch transaction. [MORE DETAILS](#swap)

- Swap SDK: The stella swap swap-sdk was based on it, but it was adapted to work with the Swap function.
    - Swap-SDK: https://github.com/stellaswap/swap-sdk/tree/main

# Screens:

ReLeaf is a comprehensive payment, savings and card solution. Integrated the SDKs and Precompiles of the Moonbeam ecosystem in a single application.

## Wallet:

As a basis for using all Moonbeam services and resources, we created a simple wallet, which allows us to view the Assets on the moonbeam network. [CODE](./ReLeaf/src/screens/main/tabs/tab1.js)

<img src="https://i.ibb.co/m9gwVzc/vlcsnap-2024-05-05-13h55m07s508.png" width="32%">

In turn, this tab integrates the contract of [ReLeafBalances](./Contracts/ReLeafBalances.sol), which allows us to obtain all the balances of all the X-Tokens in the Moonbeam ecosystem from a single RPC Call.

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

With the send function, we can send native tokens or ERC20 tokens in batch, which means that all transactions will be executed in the same transaction. [CODE](./ReLeaf/src/screens/sendWallet/sendWallet.js)

<img src="https://i.ibb.co/XV1BYJQ/vlcsnap-2024-05-05-13h55m25s064.png" width="32%"> <img src="https://i.ibb.co/K6ztVJz/vlcsnap-2024-05-05-13h55m28s247.png" width="32%"> <img src="https://i.ibb.co/xJXSbNg/vlcsnap-2024-05-05-13h55m31s607.png" width="32%">

It is very important to check that all transactions are executed through the [Batch Precompile](https://docs.moonbeam.network/builders/pallets-precompiles/precompiles/batch/) Contract of Moonbeam. 

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

With this screen, you can easily show your Wallet to receive funds, whether native tokens or ERC20. [CODE](./ReLeaf/src/screens/depositWallet/depositWallet.js) 

<img src="https://i.ibb.co/bbBZ17k/vlcsnap-2024-05-05-13h55m38s880.png" width="32%">

## Swap:

Thanks to Stella Swap and the fact that its SDK is public, we can make swaps between tokens, whether native or ERC20. [CODE](./ReLeaf/src/screens/swapWallet/swapWallet.js)

<img src="https://i.ibb.co/hLSy59K/vlcsnap-2024-05-05-13h55m45s194.png" width="32%"> <img src="https://i.ibb.co/hVdNSD4/vlcsnap-2024-05-05-13h55m48s943.png" width="32%">

As mentioned before, unlike a swap from the StellaSwap platform, we can carry out the entire swap through a single transaction through [Batch Precompile](https://docs.moonbeam.network/builders/pallets-precompiles/precompiles/batch/).

    const batchData = this.batchContract.interface.encodeFunctionData(
        'batchAll',
        [
        [this.state.tokenSelected1.address, AGGREGATOR_ADDRESS],
        [],
        [allowance, swapData],
        [],
        ],
    );

## Payment: 

In this tab we intend to make it the same as using a traditional POS, this allows us to enter the amount to be charged in American dollars and to be able to make the payment with one of our virtual cards. [CODE](./ReLeaf/src/screens/paymentWallet/paymentWallet.js)

<img src="https://i.ibb.co/bdFpD77/vlcsnap-2024-05-05-13h57m37s280.png" width="32%"> <img src="https://i.ibb.co/Sm51S8r/vlcsnap-2024-05-05-13h57m44s761.png" width="32%"> <img src="https://i.ibb.co/QCSfqTK/Screenshot-20240505-150921.png" width="32%">

As you can see, since it is an AA Card, we can review the amount of money it has in all the available tokens to be able to make the payment with any of them, whether it is a native token or ERC20.

<img src="https://i.ibb.co/23f62Rb/Screenshot-20240505-150943.png" width="32%"> <img src="https://i.ibb.co/w7XWsVD/Screenshot-20240505-150949.png" width="32%"> <img src="https://i.ibb.co/0rpNSNn/Screenshot-20240505-150955.png" width="32%">

Finally, if our device has the option to print the purchase receipt, it can be printed immediately.

## Savings:

In the savings section, we can create our savings account, this account is linked to our main wallet account, meaning that our wallet will be the owner of it. [CODE](./ReLeaf/src/screens/main/tabs/tab2.js)

<img src="https://i.ibb.co/HrcQdHM/vlcsnap-2024-05-05-13h56m06s927.png" width="32%"> <img src="https://i.ibb.co/RY8bLkw/vlcsnap-2024-05-05-13h56m11s820.png" width="32%"> <img src="https://i.ibb.co/HYkCXq9/vlcsnap-2024-05-05-13h56m25s677.png" width="32%">

### Savings Protocol:

- Balanced Protocol, this protocol performs a weighted rounding according to the amount to be paid in the transaction, so that the larger the transaction, the greater the savings, in order not to affect the user. [CODE](./ReLeaf/src/utils/utils.js)

        export function balancedSavingToken(number, usd1, usd2) {
            const balance = number * usd1;
            let amount = 0;
            if (balance <= 1) {
                amount = 1;
            } else if (balance > 1 && balance <= 10) {
                amount = Math.ceil(balance);
            } else if (balance > 10 && balance <= 100) {
                const intBalance = parseInt(balance, 10);
                const value = parseInt(Math.round(intBalance).toString().slice(-2), 10);
                let unit = parseInt(Math.round(intBalance).toString().slice(-1), 10);
                let decimal = parseInt(Math.round(intBalance).toString().slice(-2, -1), 10);
                if (unit < 5) {
                unit = '5';
                decimal = decimal.toString();
                } else {
                unit = '0';
                decimal = (decimal + 1).toString();
                }
                amount = intBalance - value + parseInt(decimal + unit, 10);
            } else if (balance > 100) {
                const intBalance = parseInt(Math.floor(balance / 10), 10);
                amount = (intBalance + 1) * 10;
            }
            return new Decimal(amount).sub(new Decimal(balance)).div(usd2).toNumber();
        }

- Percentage protocol, unlike the previous protocol, this one aims to always save a percentage selected in the UI. [CODE](./ReLeaf/src/utils/utils.js)

        export function percentageSaving(number, percentage) {
            return number * (percentage / 100);
        }

## Cards:

Finally, in the cards section, we can create a virtual card, which will help us make payments without the need for our wallet directly with a physical card in any POS terminal with ReLeaf. [CODE](./Cloud%20Functions/Add%20Card/index.js)

<img src="https://i.ibb.co/P9nb73m/vlcsnap-2024-05-05-13h56m32s908.png" width="32%"> <img src="https://i.ibb.co/28VTxY7/vlcsnap-2024-05-05-13h56m38s244.png" width="32%"> <img src="https://i.ibb.co/NKp3rgR/vlcsnap-2024-05-05-13h56m47s323.png" width="32%">

This AA Card has as its owner a wallet completely controlled by our backend in Google Cloud. However, the only way to make payments from this card is through the physical card. And all transactions are encrypted using SHA256.[CODE](./Cloud%20Functions/Card%20Transaction/index.js)
