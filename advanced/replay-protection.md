# Replay protection

The Forwarder contract implements a two dimensional nonce from [EIP 2585](https://github.com/ethereum/EIPs/issues/2585) .

We think it's the less restrictive approach. It allows ordered but also concurrent meta-transactions. It provides different channels for nonce management. If you don't need concurrency, just use one channel. If you need concurrent transactions, use different channels. An extra gas fee will be charged the first time you use a channel.

In this implementation, the nonce is an `uint256` that is split in two 128 bit values. The higher bits represent the channel ID while the lower bits represent the nonce in the channel. The nonce to be sent is equal to the current nonce of a channel. For every use the current nonce get increased by one.

```javascript
mapping(address => mapping(uint128 => uint128)) public nonces;

    function checkAndUpdateNonce(address signer, uint256 nonce) internal returns (bool) {
        uint128 channelId = uint128(nonce % 2**128);
        uint128 channelNonce = uint128(nonce / 2**128);

        uint128 currentNonce = nonces[signer][channelId];
        if (channelNonce == currentNonce) {
            nonces[signer][channelId]++;
            return true;
        }
        return false;
    }
```

