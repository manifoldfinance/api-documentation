---
title: quickstart guide
description: overview of eth_Call/sendRawTransaction & Revert Reasons
---

# Quickstart

- [Quickstart](#quickstart)
  * [Revert reason](#revert-reason)
  * [Enabling revert reason](#enabling-revert-reason)
  * [Where is the revert reason included](#where-is-the-revert-reason-included)
  * [Revert reason format](#revert-reason-format)
  * [Dapp support](#dapp-support)
  * [Creating and sending transactions](#creating-and-sending-transactions)
  * [`eth_call` vs `eth_sendRawTransaction`](#-eth-call--vs--eth-sendrawtransaction-)



## Revert reason

In smart contracts, the
[`revert`](https://solidity.readthedocs.io/en/v0.5.10/control-structures.html#revert) operation
triggers an exception to flag an error and revert the current call. The EVM passes back to the
client an optional string message containing information about the error.

!!! example

    ```sol
    pragma solidity >=0.5.0 <0.7.0;

    contract VendingMachine {
        function buy(uint amount) public payable {
            if (amount > msg.value / 2 ether)
                revert("Not enough Ether provided");
            // Alternative way to do it:
            require(
                amount <= msg.value / 2 ether,
                "Not enough Ether provided."
            );
            // Perform the purchase.
        }
    }
    ```

## Enabling revert reason

Use the [`--revert-reason-enabled`](../../Reference/CLI/CLI-Syntax.md#revert-reason-enabled)
command line option to include the revert reason in the transaction receipt in Hyperledger Besu.

!!! caution

    Enabling revert reason may use a significant amount of memory. We do not recommend enabling
    revert reason when connected to public Ethereum networks.

## Where is the revert reason included

With revert reason enabled, the transaction receipt returned by
[`eth_getTransactionReceipt`](../../Reference/API-Methods.md#eth_gettransactionreceipt) includes
the revert reason as an ABI-encoded string.

!!! important

    The revert reason is not included in the transactions receipt's root hash. Not including the
    revert reason in the transactions receipt's root hash means the revert reason is only available
    to nodes that execute the transaction when importing the block. That is, the revert reason is
    not available if using fast synchronization
    ([`--sync-mode=FAST`](../../Reference/CLI/CLI-Syntax.md#sync-mode)).

!!! example

    ```json
    {
      "jsonrpc": "2.0",
      "id": 1,
      "result": {
         "blockHash": "0xe7212a92cfb9b06addc80dec2a0dfae9ea94fd344efeb157c41e12994fcad60a",
         "blockNumber": "0x50",
         "contractAddress": null,
         "cumulativeGasUsed": "0x5208",
         "from": "0x627306090abab3a6e1400e9345bc60c78a8bef57",
         "gasUsed": "0x5208",
         "logs": [],
         "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
         "status": "0x1",
         "to": "0xf17f52151ebef6c7334fad080c5704d77216b732",
         "transactionHash": "0xc00e97af59c6f88de163306935f7682af1a34c67245e414537d02e422815efc3",
         "transactionIndex": "0x0",
         "revertReason":"0x08c379a00000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000001a4e6f7420656e6f7567682045746865722070726f76696465642e000000000000"
       }
    }
    ```

## Revert reason format

As described in the [Solidity documentation], the transaction receipt includes the revert reason as
an ABI-encoded string consisting of:

```bash
0x08c379a0                                                         // Function selector for Error(string)
0x0000000000000000000000000000000000000000000000000000000000000020 // Data offset
0x000000000000000000000000000000000000000000000000000000000000001a // String length
0x4e6f7420656e6f7567682045746865722070726f76696465642e000000000000 // String data
```

!!! example "Example of revert reason string for \"Not enough Ether provided.\""

    ```bash
    "0x08c379a00000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000001a4e6f7420656e6f7567682045746865722070726f76696465642e000000000000"
    ```

## Dapp support

Client libraries, such as web3j, do not support extracting the revert reason from the transaction
receipt. To extract the revert reason your Dapp must interact directly with Besu using a custom
JSON -> Object converter.

<!-- Links -->
[Solidity documentation]: https://solidity.readthedocs.io/en/v0.5.10/control-structures.html#revert


## Creating and sending transactions

You can send signed transactions using the
[`eth_sendRawTransaction`](../../mev/eth_sendRawTransaction/README.md) JSON-RPC API
method.

Signed transactions can be simple value transfers, contract creation, or contract invocation. Set
the maximum transaction fee for transactions using the [`--rpc-tx-feecap`](../../Reference/CLI/CLI-Syntax.md#rpc-tx-feecap)
CLI option.

[Use client libraries](../Develop-Dapps/Client-Libraries.md) to create and send a signed raw transaction to transfer
Ether and create a smart contract.

!!! warning "Private keys"

    Do not use the accounts from the examples on Mainnet or any public network except for testing.
    The private keys are displayed which means the accounts are not secure.

    All accounts and private keys in the examples are from the `dev.json` genesis file in the
    [`/besu/config/src/main/resources`](https://github.com/hyperledger/besu/tree/master/config/src/main/resources)
    directory.

    In production environments avoid exposing your private keys by creating signed transactions
    offline, or use [EthSigner](https://docs.ethsigner.consensys.net/) to isolate your private keys
    and sign transactions with
    [`eth_sendTransaction`](https://docs.ethsigner.consensys.net/Using-EthSigner/Using-EthSigner/#eth_sendtransaction).

!!! tip

    Libraries such as [web3j](https://github.com/web3j/web3j) or
    [ethereumj](https://github.com/ethereum/ethereumj) and tools such as
    [MyCrypto](https://mycrypto.com/) can also create signed transactions.

## `eth_call` vs `eth_sendRawTransaction`

You can interact with contracts using [`eth_call`](../../Reference/API-Methods.md#eth_call) or
[`eth_sendRawTransaction`](../../Reference/API-Methods.md#eth_sendrawtransaction). The table below
compares the characteristics of both calls.

| `eth_call`                                                     | `eth_sendRawTransaction`                                                                                                      |
|----------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------|
| Read-only                                                      | Write                                                                                                                         |
| Invokes contract function locally                              | Broadcasts to the network                                                                                                     |
| Does not change state of blockchain                            | Updates the blockchain (for example, transfers ether between accounts)                                                        |
| Does not consume gas                                           | Requires gas                                                                                                                  |
| Synchronous                                                    | Asynchronous                                                                                                                  |
| Returns the value of a contract function available immediately | Returns transaction hash only. A block might not include all possible transactions (for example, if the gas price is too low). |
