---
description: Let's make some native assets on Cardano ❤️✨
---

# Cardano Native Asset and NFT Tutorial

## Who is this guide for?

* For people who want to make NFT's or Native Assets on Cardano
* For people who know about Cardano

## Benefits of NFT's on Cardano

* Low transaction fees
* Native on the blockchain

## Prerequisites

* cardano-node / cardano-cli set up on local machine

{% hint style="info" %}
[Here](intermediate-guide/pi-pool-tutorial/pi-node/) is an easy to follow tutorial we made to get a Cardano-node/cli running on mainnet
{% endhint %}

* Make sure you have a Cardano node running and fully synced to the database
* Make sure node.js installed

```bash
#Copy/Paste this into your terminal if node.js is not installed
curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
sudo apt-get install -y nodejs
```

{% hint style="warning" %}
We made this tutorial on Raspberry-Pi ARM machines so make sure to download the **correct** node.js for your **local machine/CPU and OS**.
{% endhint %}

### Verify everything is set up properly on our machine ⚙️

```bash
#Copy/paste into terminal window
cardano-cli version; cardano-node version
```

 Your output should look like this 👇

```bash
cardano-cli 1.26.1 - linux-aarch64 - ghc-8.10
git rev 0000000000000000000000000000000000000000
cardano-node 1.26.1 - linux-aarch64 - ghc-8.10
git rev 0000000000000000000000000000000000000000
```

#### Verify our node.js version is correct and is on v14.16.0

```bash
#Copy/paste into terminal window
node -v
```

```bash
v14.16.0
```

#### Video Walk-through:

{% embed url="https://youtu.be/oP3jZyPxB-I" %}



## Create our project directory and initial setup

```bash
# check for $NODE_HOME
echo $NODE_HOME

# if the above echo didn't return anything, you need to set a $NODE_HOME
# or use a static path for the CARDANO_NODE_SOCKET_PATH location

export NODE_HOME="/home/ada/pi-pool"

# change this to match where your cardano-node socket is located
export CARDANO_NODE_SOCKET_PATH="$NODE_HOME/db/socket"

mkdir cardano-minter
cd cardano-minter
npm init -y #creates package.json)
npm install cardanocli-js --save
```

1. **Copy the Cardano node genesis latest build number from the IOHK hydra website**
   * [https://hydra.iohk.io/job/Cardano/cardano-node/cardano-deployment/latest-finished/download/1/index.html](https://hydra.iohk.io/job/Cardano/cardano-node/cardano-deployment/latest-finished/download/1/index.html)
2. **Create a bash shell script to Download the latest Genesis config file needed**

```bash
nano fetch-config.sh
```

{% tabs %}
{% tab title="MAINNET" %}
```bash
#NODE_BUILD_NUM may be different
NODE_BUILD_NUM=5822084
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/mainnet-shelley-genesis.json
```
{% endtab %}

{% tab title="TESTNET" %}
```bash
NODE_BUILD_NUM=5822084
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/testnet-shelley-genesis.json
```
{% endtab %}
{% endtabs %}

**Now we need to give permissions to our new script to execute then we will run our script and download the genesis files.**

```bash
chmod +x fetch-config.sh
./fetch-config.sh
```

### Next, we make our src folder/directory and then create the Cardano client.

```bash
mkdir src; cd src
nano cardano.js
```

{% hint style="info" %}
If you are using testnet make sure you have the correct testnet-magic version number. You can find the current testnet version [here](https://docs.cardano.org/projects/cardano-node/en/latest/stake-pool-operations/getConfigFiles_AND_Connect.html).
{% endhint %}

{% tabs %}
{% tab title="MAINNET" %}
```javascript
const Cardano = require("cardanocli-js");

const cardano = new Cardano({
    network: "mainnet",
    dir: __dirname + "/../",
    shelleyGenesisPath: __dirname + "/../mainnet-shelley-genesis.json",
});

module.exports = cardano;
```
{% endtab %}

{% tab title="TESTNET" %}
```javascript
const Cardano = require("cardanocli-js")

const cardano = new Cardano({
    network: "testnet-magic 1097911063",
    dir: __dirname + "/../",
    shelleyGenesisPath: __dirname + "/../testnet-shelley-genesis.json"
});

module.exports = cardano;
```
{% endtab %}
{% endtabs %}

#### _Video Walk-through_ :

{% tabs %}
{% tab title="Project\'s Initial Setup" %}
{% embed url="https://youtu.be/Xkx9vdibbq0" %}
{% endtab %}

{% tab title="Fetching the genesis files" %}
{% embed url="https://youtu.be/X5cRGA0qyQE" %}
{% endtab %}

{% tab title="Create the Cardano client" %}
{% embed url="https://youtu.be/-fnaF3FWL3k" %}
{% endtab %}
{% endtabs %}

## Create a local wallet

```bash
nano create-wallet.js
```

```javascript
const cardano = require('./cardano')

const createWallet = (account) => {
  cardano.addressKeyGen(account);
  cardano.stakeAddressKeyGen(account);
  cardano.stakeAddressBuild(account);
  cardano.addressBuild(account);
  return cardano.wallet(account);
};

createWallet("ADAPI")
```

```bash
cd cardano-minter
node src/create-wallet.js
```

#### Verify balance wallet balance is Zero, then we fund the wallet

* **First, we need to create a get-balance.js script**

```bash
cd cardano-minter/src; nano get-balance.js
```

```javascript
// create get-balance.js
const cardano = require('./cardano')

const sender = cardano.wallet("ADAPI");

console.log(
    sender.balance()
)
```

* **Now, Check the balance of our wallet.**

```text
cd ..
node src/get-balance.js
```

* We can go ahead and send some funds \(ADA\) into our wallet we created, wait a few minutes, and then check the balance again to make sure the transaction was successful.

{% hint style="info" %}
If you are using testnet you must get your tADA from the testnet faucet [here](https://developers.cardano.org/en/testnets/cardano/tools/faucet/).
{% endhint %}

#### _Video Walk-through_ :

{% tabs %}
{% tab title="Create local wallet" %}
{% embed url="https://youtu.be/a8uWUc0O3DE" %}
{% endtab %}

{% tab title="Sending funds to our wallet" %}
{% embed url="https://youtu.be/Mm1ZOciiNaE" %}
{% endtab %}
{% endtabs %}



## Mint our Native-Asset/NFT on Cardano

Before we proceed to mint our Native Asset we must have a few things taken care of. We need to first get our "asset" onto our [IPFS](https://ipfs.io/#install) node and generate the IPFS link. If you do not know about IPFS or what it actually does we recommend having a read through the documentation [here](https://docs.ipfs.io/) or watching this [video](https://www.youtube.com/watch?v=5Uj6uR3fp-U).

Since we are using an image file to be our asset we should upload a smaller thumbnail-sized version of our image \(ideally less than 1MB\). This will be used on sites like [pool.pm](https://pool.pm) to display our assets nicely in our wallets. We then upload the full-size image as our source image.

* [ ] Download [IPFS](https://ipfs.io/#install)
* [ ] Upload your asset's files to IPFS
* [ ] Get our image thumbnail IPFS link
* [ ] Get the src IPFS link

#### For reference:

* **image \(thumbnail version\) - ipfs://QmQqzMTavQgT4f4T5v6PWBp7XNKtoPmC9jvn12WPT3gkSE**
* **src \(full-size version\) - ipfs://Qmaou5UzxPmPKVVTM9GzXPrDufP55EDZCtQmpy3T64ab9N**

### Create our mint-asset.js script

This script has three main components:

1. **Generate policy id**
2. **Define your metadata**
3. **Create mint transaction**

```javascript

const fs = require("fs");
const cardano = require("./cardano");

// 1. Get the wallet
// 2. Define mint script
// 3. Create POLICY_ID
// 4. Define ASSET_NAME
// 5. Create ASSET_ID
// 6. Define metadata
// 7. Define transaction
// 8. Build transaction
// 9. Sign transaction
// 10. Submit transaction

const buildTransaction = (tx) => {
  const raw = cardano.transactionBuildRaw(tx);
  const fee = cardano.transactionCalculateMinFee({
    ...tx,
    txBody: raw,
  });
  tx.txOut[0].amount.lovelace -= fee;
  return cardano.transactionBuildRaw({ ...tx, fee });
};

const signTransaction = (wallet, tx, script) => {
  return cardano.transactionSign({
    signingKeys: [wallet.payment.skey, wallet.payment.skey],
    scriptFile: script,
    txBody: tx,
  });
};

const wallet = cardano.wallet("ADAPI");

const mintScript = {
  keyHash: cardano.addressKeyHash(wallet.name),
  type: "sig",
};

const POLICY_ID = cardano.transactionPolicyid(mintScript);
const ASSET_NAME = "BerrySpaceGreen";
const ASSET_ID = POLICY_ID + "." + ASSET_NAME;

const metadata = {
  721: {
    [POLICY_ID]: {
      [ASSET_NAME]: {
        name: ASSET_NAME,
        image: "ipfs://QmQqzMTavQgT4f4T5v6PWBp7XNKtoPmC9jvn12WPT3gkSE",
        description: "Super Fancy Berry Space Green NFT",
        type: "image/png",
        src: "ipfs://Qmaou5UzxPmPKVVTM9GzXPrDufP55EDZCtQmpy3T64ab9N",
        authors: ["PIADA", "SBLYR"],
      },
    },
  },
};

const tx = {
  txIn: wallet.balance().utxo,
  txOut: [
    {
      address: wallet.paymentAddr,
      amount: { ...wallet.balance().amount, [ASSET_ID]: 1 },
    },
  ],
  mint: [{ action: "mint", amount: 1, token: ASSET_ID }],
  metadata,
  witnessCount: 2,
};

const raw = buildTransaction(tx);
const signed = signTransaction(wallet, raw, mintScript);
const txHash = cardano.transactionSubmit(signed);
console.log(txHash);
```

* **Run the minting script, then wait a few moments to check the balance in our wallet**

```text
cd ..
node src/mint-asset.js
```

{% tabs %}
{% tab title="Mint asset" %}
{% embed url="https://youtu.be/qTzLgMyJC7s" %}
{% endtab %}
{% endtabs %}

## Sending your NFT back to Daedulus or Yoroi wallet



Now we must create a new script to send our newly minted NFT to a wallet.

```javascript
cd cardano-minter/src
nano send-back-asset-to-wallet.js
```

There are few main parts we have to this script in order to send the asset:

1. Get the wallet
2. Define the transaction
3. Build the transaction
4. Calculate the fee
5. Pay the fee by subtracting it from the sender's utxo
6. Build the final transaction
7. Sign the transaction
8. Submit the transaction

```javascript
const cardano = require("./cardano");

const sender = cardano.wallet("ADAPI");

console.log(
  "Balance of Sender wallet: " +
    cardano.toAda(sender.balance().amount.lovelace) +
    " ADA"
);

const receiver =
  "addr1qym6pxg9q4ussr96c9e6xjdf2ajjdmwyjknwculadjya488pqap23lgmrz38glvuz8qlzdxyarygwgu3knznwhnrq92q0t2dv0";

const txInfo = {
  txIn: cardano.queryUtxo(sender.paymentAddr),
  txOut: [
    {
      address: sender.paymentAddr,
      amount: {
        lovelace: sender.balance().amount.lovelace - cardano.toLovelace(1.5),
      },
    },
    {
      address: receiver,
      amount: {
        lovelace: cardano.toLovelace(1.5),
        "ad9c09fa0a62ee42fb9555ef7d7d58e782fa74687a23b62caf3a8025.BerrySpaceGreen": 1,
      },
    },
  ],
};

const raw = cardano.transactionBuildRaw(txInfo);

const fee = cardano.transactionCalculateMinFee({
  ...txInfo,
  txBody: raw,
  witnessCount: 1,
});

//pay the fee by subtracting it from the sender utxo
txInfo.txOut[0].amount.lovelace -= fee;

//create final transaction
const tx = cardano.transactionBuildRaw({ ...txInfo, fee });

//sign the transaction
const txSigned = cardano.transactionSign({
  txBody: tx,
  signingKeys: [sender.payment.skey],
});

//subm transaction
const txHash = cardano.transactionSubmit(txSigned);
console.log("TxHash: " + txHash);
```

```javascript
cd ..
node src/send-back-asset-to-wallet.js
```

### Final Steps to view your NFT

1. View your nft in your wallet
2. View your asset on cardanoassets.com
3. View your asset on pool.pm \(see the actual picture\)
4. Show the original minting metadata
5. Open the src and image ipfs links in your browser to prove that it worked

#### _Video Walk-through:_

{% embed url="https://youtu.be/awxVkFbWoKM" %}
