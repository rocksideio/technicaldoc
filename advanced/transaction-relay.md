# Transaction relay

To relay your transactions we use the concept of meta-transactions. Meta-transactions are based on the principle of off-chain message signing for on-chain use. A user signs a message representing its transaction intent. The message is then sent to Rockside. There it gets wrapped in a new transaction to be sent and executed on chain by a smart contract.

![Relay overview](https://raw.githubusercontent.com/rocksideio/technicaldoc/master/images/tx-relay-overview.png)

## User signs its message and sends it to Rockside

The parameters to be included on the signed message are:

* **signer**: address of the account who signed the message. The user who want to interact with the destination contract.
* **to**: if the destination contract has to interact with another account \(contract or EOA\), this field can be used \(for example to transfer tokens\).
* **data**: bytes to be executed.
* **nonce**: nonce value used by the signer. It's a 2 dimensional nonce represented as a 256 bit integer split in two. See [replay protection page](replay-protection.md)

At Rockside we follow [EIP-712](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-712.md) to structure the relay message.

When the message is created and signed, it's sent to Rockside with a chosen speed of inclusion in the blockchain.

### Rockside validates the transaction and sends it to the Forwarder

1. **Accept the transaction**: Rockside use EthGasStation as a reference for the gas prices. Depending on your given gas price limit, your requested speed and the current market gas prices, we decide whether or not to relay your transaction. We also verify that the Forwarder has enough ether to refund Rockside for the gas used by the transaction.
2. **Choose the appropriate EOA**: Rockside manages different pools of EOA to send transactions. A pool of EOA for each available speed. This way, we guarantee that a "fast" transaction will not be slowed down by a transaction with a "safelow" speed. It's like on the highway, each transaction has its own queue depending on its speed.
3. **The message is included within a transaction**: Using the corresponding EOA a transaction containing the signed message of the user is sent to the user's Forwarder. The gas price used by Rockside is in accordance with the speed requested.

The source code of Forwarder contract is available[ on Github](https://github.com/rocksideio/contracts/blob/master/contracts/Forwarder.sol).

### The Forwarder validates the message and calls the destination contract

1. **Message signature validation**: The Forwarder verifies that the signature corresponds to the signer and the parameters of the transaction.
2. **Check and update nonce**: To avoid replay attacks, the forwarder verifies that the nonce was not already used. Once done, the current nonce is incremented.
3. **Call the destination contract**: When all verifications are done, the destination contract is called with the parameters of the transactions.
4. **Refund Rockside relayer**: An amount of ether corresponding to the gas consumed and the gas price used \(limited by the gas price limit\) is sent from the Forwarder to the Rockside Relayer.

### The transaction is executed

The destination contract [is called with "data+signer"](https://github.com/rocksideio/contracts/blob/master/contracts/Forwarder.sol#L110). To be compatible with Rockside's forwarder, the destination contract must use a special function instead of the usual `msg.sender`

```text
function _msgSender() internal view returns (address ret) {
        address sender = msg.sender;
        if (msg.data.length >= 24 && msg.sender == authorizedForwarder) {
            assembly {
                sender := shr(96,calldataload(sub(calldatasize(),20)))
            }
        }
        return sender;
    }
```

This function is used for your contract to know if it was called by a meta transaction or a normal transaction.

You can see an example of implementation in [our SmartWallet contract](https://github.com/rocksideio/contracts/blob/master/contracts/SmartWallet.sol).

