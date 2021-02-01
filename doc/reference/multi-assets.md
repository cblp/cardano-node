# Multi-asset support

Starting from Mary (the ledger development upgrade) and onwards, Cardano supports the [multi-asset](https://hydra.iohk.io/job/Cardano/cardano-ledger-specs/specs.shelley-ma/latest/download-by-type/doc-pdf/shelley-ma) feature, which is also referred to as *native tokens*. Multi-asset support extends the existing accounting infrastructure defined in the ledger model (originally designed for processing ada-only transactions) to accommodate transactions using various assets. 

## What is a multi-asset?

Multi-assets include ada and a variety of user-defined (custom) token types that can be created and transacted natively. A single tx output, for example, can have a mixture of ada and native tokens.

Native support offers distinct advantages for developers as there is no need to create smart contracts to mint or burn custom tokens. This removes a layer of added complexity and potential for manual errors since the ledger handles all token-related functionality. 

Each token is identified by its asset ID (a unique identifier for a collection of tokens), which consists of a policy ID (hash of the minting policy) and an asset name. Tokens that have the same asset ID are fungible with each other and are not fungible with tokens that have a different asset ID. 

## Minting a multi-asset token

### Step 1 - create a script

First, generate the keys that you require witnesses from using the
`cardano-cli address key-gen` command. Until we hard fork to the Mary era, we will
use a simple script to generate the policy ID. 

*Note that you can use a Plutus script when it is available.*

Construct a simple script in JSON syntax
as described [here](./simple-scripts.md). For this example, we will describe the process using the following script:

```json
 {
  "keyHash":$KEYHASH,
  "type": "sig"
 }
```

where `$KEYHASH` is generated as follows:

```bash
cardano-cli address key-hash --payment-verification-key-file policy.vkey
```

Similarly, generate the policy ID:

```bash
cardano-cli transaction policyid --script-file policy.script
```

### Step 2 - construct, sign, and submit the transaction

Construct the tx body and specify the multi-asset you would like to mint. Note that you must spend at least the minimum UTxO value and it is essential to hold ada (besides other currencies) to transfer multi-asset tokens between addresses. In the tx body below we mint and spend 5 tokens of a particular multi-asset: 

```bash
cardano-cli transaction build-raw \
            --mary-era \
            --fee 0 \
            --tx-in $TXIN \
            --tx-out $ADDR + 5 $POLICYID.couttscoin\
            --mint 5 $POLICYID.yourassetname \
            --out-file txbody
```

Sign the transaction with the appropriate signing keys and include the script:

```bash
cardano-cli transaction sign \
            --signing-key-file txin.skey \
            --signing-key-file policy.skey \
            --script-file $SCRIPT \
            --testnet-magic 42 \
            --tx-body-file  txbody \
            --out-file      tx
```

Submit the transaction:

```bash
cardano-cli transaction submit --tx-file  tx --testnet-magic 42
```

## Burning a multi-asset token

Create tx body that will burn 5 couttscoins:

```bash
cardano-cli transaction build-raw \
            --mary-era \
            --fee 0 \
            --tx-in $TXIN \
            --tx-out $TXOUT\
            --mint -5 $POLICYID.couttscoin \
            --out-file txbodyburn
```

Sign the transaction:

```bash
cardano-cli transaction sign \
            --signing-key-file txin.skey \
            --signing-key-file policy.skey \
            --script-file $SCRIPT \
            --testnet-magic 42 \
            --tx-body-file  txbodyburn \
            --out-file      txburn
```

Submit the transaction:

```bash
cardano-cli transaction submit --tx-file txburn --testnet-magic 42
```
