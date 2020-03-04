## Script Usage:

## Disclaimer:

Using own risk!

### Prerequisites

Contracts must be compiled and, if working on any non-local network, deployment addresses must be injected into built contracts.
To do so, run:

```
npx truffle compile
yarn run networks-inject
```

Setup the env variables:

```
export PK=<Your Key>
export GAS_PRICE_GWEI=8
export NETWORK_NAME=<network>
export MASTER_SAFE=<master safe>

```

### Confirming multisig orders with Metamask

These script require to set up a "Master Safe" created through the interface (here)[https://rinkeby.gnosis-safe.io] with an owner key under your control. This Master Safe should have an additional owner `0x90F8bf6A479f320ead074411a4B0e7944Ea8c9C1`, known as the "Proposer" account. The following scripts will use this account to propose transactions to the interface and this implies that the mnemonic phrase for this "Proposer" account is stored in plain text within this project.
In order to have a save setup, make sure that _your Master Safe always requires one more signature than just the signature of the Proposer account to send a transaction_. Otherwise, everyone can steal the funds from your account!

For the signing process, note that the metamask underestimates the gas consumption. Just make sure that the gas limit of your transaction is sufficient. Since there is no proper way to estimate gas, setting the gas limit to a value higher than 4_000_000 is a practical, yet insecure way.

### Deploy Safes

```js
truffle exec scripts/deploy_safes.js --masterSafe=$MASTER_SAFE --fleetSize=2 --network $NETWORK_NAME
```

### Place Orders

Requires that Master and Slave Safes are already deployed.

```js
truffle exec scripts/bracket_orders.js --targetToken=1 --stableToken=7 --targetPrice 270 --masterSafe=$MASTER_SAFE --slaves=0xb947de73ADe9aBC6D57eb34B2CC2efd41f646636,0xfA4a18c2218945bC018BF94D093BCa66c88D3c40 --network=$NETWORK_NAME
```

### Transfer-Approve-Deposit

```js
truffle exec scripts/transfer_approve_deposit.js --masterSafe=$MASTER_SAFE --depositFile="./data/depositList.json" --network=$NETWORK_NAME
```

### Withdrawing

To withdraw funds, all withdrawals have to be specified in a file with the following format:

```
    {
        "amount": "100000000000000000",
        "tokenAddress": "0xc778417e063141139fce010982780140aa0cd5ab",
        "userAddress": "0xfA4a18c2218945bC018BF94D093BCa66c88D3c40"
    }
]
```

The script can automatically determine the amount, instead of having to specify it on the file.
This is achieved by adding the flag `--allTokens` to the withraw command. This is possible in any of the following commands.

Withdrawing is a two-step process: first a withdraw must be requested on the exchange, then the withdraw is executed and at the same time the funds are sent back to the master Safe.

```js
truffle exec scripts/withdraw.js --requestWithdraw --masterSafe=$MASTER_SAFE --withdrawals="./data/depositList.json" --network=$NETWORK_NAME
```

```js
truffle exec scripts/withdraw.js --withdraw --transferBackToMaster --masterSafe=$MASTER_SAFE --withdrawals="./data/depositList.json" --network=$NETWORK_NAME
```

The latter instruction can be split into two independent units, if needed: withdrawing from the exchange to the traders and transferring funds from the traders to the master Safe.

```js
truffle exec scripts/withdraw.js --withdraw --masterSafe=$MASTER_SAFE --withdrawalsFromDepositFile="./data/depositList.json" --network=$NETWORK_NAME
```

```js
truffle exec scripts/withdraw.js --transferBackToMaster --masterSafe=$MASTER_SAFE --withdrawalsFromDepositFile="./data/depositList.json" --network=$NETWORK_NAME
```

### Full Cycle Test

To begin, you must have an existing "Master Safe" created through the interface (here)[https://rinkeby.gnosis-safe.io] for convienience, add an additional owner `0x90F8bf6A479f320ead074411a4B0e7944Ea8c9C1`, known as the "Proposer" account.

Then, export the address like this:

```
export MASTER_SAFE=0xb2162C8868AB135604270c92ed1faFA47b2BB50B
```

Set all the other env variables.

2. Create subsafes via

```js
truffle exec scripts/deploy_safes.js --masterSafe=$MASTER_SAFE --fleetSize=20 --network $NETWORK_NAME
```

The deployed safes addresses will be displayed in the terminal and should be recorded somewhere. Otherwise, they will be difficult to recover (at least untill we can deploy them all in a single transaction).

2. Place Bracket Orders

Using the list of safes just deployed and whatever values you would like for the other parameters.

```js
truffle exec scripts/bracket_orders.js --targetToken=1 --stableToken=7 --targetPrice 278 --masterSafe=$MASTER_SAFE --slaves=0x0f41B6BaF8202512e62CCc9866D27032D9DCFcD1,0x1d8A3D8d21466d36DDCa5432FD8b5d95956023B5,0x97f87462cC8738E1AFf8656A0C1D499e28d17E3b,0xF3b3609BEb25CB725377AaF06eBf50ff856F5C64,0x5e95f9a4f4Ed1323Dd489bDfd10bdd718e4d4720,0x194d09D906846aeF3402c5eab775bF8e6d6A9556,0xe635149d28302891377168964f873728175E8976,0x8beF6f835628688bDaC3Db28030769F5BCd37240,0xFEFC78e2363dCD8f9699F9336b14B292B29cf98e,0x8932112aEB83C7f7AebE9c83Beb62BCe5ad83275,0x90FeeD152232eD3fe0dAe28f2BE3651e38208E60,0xE6f636BD71e42F8D4EB17F47b216E945E0C156aE,0x5961D9411e6722BD299a5C36a7A5CDD3fc5Ec35E,0x52863CB34be1bE58B199281a6623D259BB670950,0x88580dd3dF04Eb9418D9ECC694FBc95786189cFE,0x185076bcba4aBD58F8d07C7276B7194Ec124Ea55,0x3A8CE5F3186C0E116d412c80D1e56b9778182a1A,0x0703B45ca5172016aF1f9fdF43f80bDc9088b542,0xBd7982cE05fe5484448b775E2FD017203Cd9aE90,0x5F1D32eFF4E25c4FD8C57d8cf3F8251eFB82bEC2 --network=$NETWORK_NAME
```

This will reult in the following text displayed in your terminal. Note that you will be prompted before the transaction is sent.

```
Preparing order transaction data
Batch Exchange 0xC576eA7bd102F7E476368a5E98FA455d1Ea34dE2
Lowest-Highest Limit 222.4-333.59999999999997
Constructing bracket trading strategy order data based on valuation 278 [object Object] per WETH
Safe 0 - 0x0f41B6BaF8202512e62CCc9866D27032D9DCFcD1:
  Buy  WETH with DAI at 222.4
  Sell WETH for  DAI at 227.96
Safe 1 - 0x1d8A3D8d21466d36DDCa5432FD8b5d95956023B5:
  Buy  WETH with DAI at 227.96
  Sell WETH for  DAI at 233.52
Safe 2 - 0x97f87462cC8738E1AFf8656A0C1D499e28d17E3b:
  Buy  WETH with DAI at 233.52
  Sell WETH for  DAI at 239.07999999999998
Safe 3 - 0xF3b3609BEb25CB725377AaF06eBf50ff856F5C64:
  Buy  WETH with DAI at 239.07999999999998
  Sell WETH for  DAI at 244.64
Safe 4 - 0x5e95f9a4f4Ed1323Dd489bDfd10bdd718e4d4720:
  Buy  WETH with DAI at 244.64
  Sell WETH for  DAI at 250.2
Safe 5 - 0x194d09D906846aeF3402c5eab775bF8e6d6A9556:
  Buy  WETH with DAI at 250.2
  Sell WETH for  DAI at 255.76
Safe 6 - 0xe635149d28302891377168964f873728175E8976:
  Buy  WETH with DAI at 255.76
  Sell WETH for  DAI at 261.32
Safe 7 - 0x8beF6f835628688bDaC3Db28030769F5BCd37240:
  Buy  WETH with DAI at 261.32
  Sell WETH for  DAI at 266.88
Safe 8 - 0xFEFC78e2363dCD8f9699F9336b14B292B29cf98e:
  Buy  WETH with DAI at 266.88
  Sell WETH for  DAI at 272.44
Safe 9 - 0x8932112aEB83C7f7AebE9c83Beb62BCe5ad83275:
  Buy  WETH with DAI at 272.44
  Sell WETH for  DAI at 278
Safe 10 - 0x90FeeD152232eD3fe0dAe28f2BE3651e38208E60:
  Buy  WETH with DAI at 278
  Sell WETH for  DAI at 283.56
Safe 11 - 0xE6f636BD71e42F8D4EB17F47b216E945E0C156aE:
  Buy  WETH with DAI at 283.56
  Sell WETH for  DAI at 289.12
Safe 12 - 0x5961D9411e6722BD299a5C36a7A5CDD3fc5Ec35E:
  Buy  WETH with DAI at 289.12
  Sell WETH for  DAI at 294.67999999999995
Safe 13 - 0x52863CB34be1bE58B199281a6623D259BB670950:
  Buy  WETH with DAI at 294.67999999999995
  Sell WETH for  DAI at 300.24
Safe 14 - 0x88580dd3dF04Eb9418D9ECC694FBc95786189cFE:
  Buy  WETH with DAI at 300.24
  Sell WETH for  DAI at 305.79999999999995
Safe 15 - 0x185076bcba4aBD58F8d07C7276B7194Ec124Ea55:
  Buy  WETH with DAI at 305.79999999999995
  Sell WETH for  DAI at 311.35999999999996
Safe 16 - 0x3A8CE5F3186C0E116d412c80D1e56b9778182a1A:
  Buy  WETH with DAI at 311.35999999999996
  Sell WETH for  DAI at 316.91999999999996
Safe 17 - 0x0703B45ca5172016aF1f9fdF43f80bDc9088b542:
  Buy  WETH with DAI at 316.91999999999996
  Sell WETH for  DAI at 322.47999999999996
Safe 18 - 0xBd7982cE05fe5484448b775E2FD017203Cd9aE90:
  Buy  WETH with DAI at 322.47999999999996
  Sell WETH for  DAI at 328.03999999999996
Safe 19 - 0x5F1D32eFF4E25c4FD8C57d8cf3F8251eFB82bEC2:
  Buy  WETH with DAI at 328.03999999999996
  Sell WETH for  DAI at 333.59999999999997
Transaction bundle size 20
Are you sure you want to send this transaction to the EVM? [yN] y
Aquiring Transaction Hash
> Signing and posting multi-send transaction request from proposer account 0x90f8bf6a479f320ead074411a4b0e7944ea8c9c1
Transaction awaiting execution in the interface https://rinkeby.gnosis-safe.io/safes/0xb2162C8868AB135604270c92ed1faFA47b2BB50B/transactions
```

Follow the link from the last line to complete signing and execution of the proposed transaction.

When propted to confirm and execute transaction, be sure to adjust the gas estimates suggested by metamask (10x)

3. Transfer-Approve-Deposit

Ensure that your `depositFile` contains all the correct information regarding correct Safe and Token addresses. For example,

```
[
    {
        "amount": "2000000000000000000",
        "tokenAddress": "0x5592ec0cfb4dbc12d3ab100b257153436a1f0fea",
        "userAddress": "0xb947de73ADe9aBC6D57eb34B2CC2efd41f646636"
    },
    {
        "amount": "1000000000000000000",
        "tokenAddress": "0xc778417e063141139fce010982780140aa0cd5ab",
        "userAddress": "    "
    }
]
```

says to deposit 2 DAI and 1 WETH to the batch exchange on behalf of `0xb947de73ADe9aBC6D57eb34B2CC2efd41f646636` and `0xfA4a18c2218945bC018BF94D093BCa66c88D3c40` respectively.

Note that, before execution of this transaction, the master safe (Fund Account) must have sufficient funds for these transfers.

```js
truffle exec scripts/transfer_approve_deposit.js --masterSafe=$MASTER_SAFE --depositFile="./data/largeDepositList.json" --network=$NETWORK_NAME
```

Should result in the following logs

```
Preparing transaction data...
Aquired Batch Exchange 0xC576eA7bd102F7E476368a5E98FA455d1Ea34dE2
Safe 0x0f41B6BaF8202512e62CCc9866D27032D9DCFcD1 receiving (from 0xb2162C8868AB135604270c92ed1faFA47b2BB50B) and depositing 1000 DAI into BatchExchange
Safe 0x1d8A3D8d21466d36DDCa5432FD8b5d95956023B5 receiving (from 0xb2162C8868AB135604270c92ed1faFA47b2BB50B) and depositing 1000 DAI into BatchExchange
Safe 0x97f87462cC8738E1AFf8656A0C1D499e28d17E3b receiving (from 0xb2162C8868AB135604270c92ed1faFA47b2BB50B) and depositing 1000 DAI into BatchExchange
Safe 0xF3b3609BEb25CB725377AaF06eBf50ff856F5C64 receiving (from 0xb2162C8868AB135604270c92ed1faFA47b2BB50B) and depositing 1000 DAI into BatchExchange
Safe 0x5e95f9a4f4Ed1323Dd489bDfd10bdd718e4d4720 receiving (from 0xb2162C8868AB135604270c92ed1faFA47b2BB50B) and depositing 1000 DAI into BatchExchange
Safe 0x194d09D906846aeF3402c5eab775bF8e6d6A9556 receiving (from 0xb2162C8868AB135604270c92ed1faFA47b2BB50B) and depositing 1000 DAI into BatchExchange
Safe 0xe635149d28302891377168964f873728175E8976 receiving (from 0xb2162C8868AB135604270c92ed1faFA47b2BB50B) and depositing 1000 DAI into BatchExchange
Safe 0x8beF6f835628688bDaC3Db28030769F5BCd37240 receiving (from 0xb2162C8868AB135604270c92ed1faFA47b2BB50B) and depositing 1000 DAI into BatchExchange
Safe 0xFEFC78e2363dCD8f9699F9336b14B292B29cf98e receiving (from 0xb2162C8868AB135604270c92ed1faFA47b2BB50B) and depositing 1000 DAI into BatchExchange
Safe 0x8932112aEB83C7f7AebE9c83Beb62BCe5ad83275 receiving (from 0xb2162C8868AB135604270c92ed1faFA47b2BB50B) and depositing 1000 DAI into BatchExchange
Safe 0x90FeeD152232eD3fe0dAe28f2BE3651e38208E60 receiving (from 0xb2162C8868AB135604270c92ed1faFA47b2BB50B) and depositing 3.6 WETH into BatchExchange
Safe 0xE6f636BD71e42F8D4EB17F47b216E945E0C156aE receiving (from 0xb2162C8868AB135604270c92ed1faFA47b2BB50B) and depositing 3.6 WETH into BatchExchange
Safe 0x5961D9411e6722BD299a5C36a7A5CDD3fc5Ec35E receiving (from 0xb2162C8868AB135604270c92ed1faFA47b2BB50B) and depositing 3.6 WETH into BatchExchange
Safe 0x52863CB34be1bE58B199281a6623D259BB670950 receiving (from 0xb2162C8868AB135604270c92ed1faFA47b2BB50B) and depositing 3.6 WETH into BatchExchange
Safe 0x88580dd3dF04Eb9418D9ECC694FBc95786189cFE receiving (from 0xb2162C8868AB135604270c92ed1faFA47b2BB50B) and depositing 3.6 WETH into BatchExchange
Safe 0x185076bcba4aBD58F8d07C7276B7194Ec124Ea55 receiving (from 0xb2162C8868AB135604270c92ed1faFA47b2BB50B) and depositing 3.6 WETH into BatchExchange
Safe 0x3A8CE5F3186C0E116d412c80D1e56b9778182a1A receiving (from 0xb2162C8868AB135604270c92ed1faFA47b2BB50B) and depositing 3.6 WETH into BatchExchange
Safe 0x0703B45ca5172016aF1f9fdF43f80bDc9088b542 receiving (from 0xb2162C8868AB135604270c92ed1faFA47b2BB50B) and depositing 3.6 WETH into BatchExchange
Safe 0xBd7982cE05fe5484448b775E2FD017203Cd9aE90 receiving (from 0xb2162C8868AB135604270c92ed1faFA47b2BB50B) and depositing 3.6 WETH into BatchExchange
Safe 0x5F1D32eFF4E25c4FD8C57d8cf3F8251eFB82bEC2 receiving (from 0xb2162C8868AB135604270c92ed1faFA47b2BB50B) and depositing 3.6 WETH into BatchExchange
Are you sure you want to send this transaction to the EVM? [yN] y
Aquiring Transaction Hash
Signing and posting multi-send transaction request from proposer account 0x90f8bf6a479f320ead074411a4b0e7944ea8c9c1
Transaction awaiting execution in the interface https://rinkeby.gnosis-safe.io/safes/0xb2162C8868AB135604270c92ed1faFA47b2BB50B/transactions
```

At the moment this script is somewhat slow because of the deposit list file format (which fetches the tokens from the EVM multiple times in order to make assertions)