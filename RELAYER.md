# Rockside Relayer

Rockside relayer is a transaction deliverery service. Give us your max gas price and your deadline, we makes sure your transaction is executed on time and at the best price.

## The problem we solve
Sometimes Ethereum transactions are stuck or lost.
To attempt to solve this problem, developers overpay transactions and must remain on-call to unblock them.
We designed a solution with a new approach to get rid of this situation

## How we do that

* **API and SDK**: Send your transactions using our API and SDK (JS, iOS)
* **Deadline and gas price limit**: Depending on your gas price limit, your deadline and current network gas prices, we accept your transaction. We make sure that your transaction is executed on time and at the best price.
* **Meta transaction**: To relay your transaction we use Meta Transaction and wraps signed message in a new transaction. By the way Rockside allows you to pay gas fees for your DApp's users. They no longer need ETH to interact with your Dapp.
* **Transaction auto replay**: We monitor your transactions to increase the price of gas when necessary to meet the deadline.
* **Pool of signers**: We manage a pool of signer with a multi-dimentionnal replay protection to garanty no stuck transaction.
* **Transaction batch** Send us more than one transaction in a call. All your transactions are executed onchain within a single transaction.
* **Tracking ID**: When sending a transaction with Rockside, you get it's transaction hash but also a trancking ID. Use it to follow the status of your transaction even it's replayed with higher gas price.


## Getting Started

### Connect to Rockside

Go to [dashboard.rockside.io](https://dashboard.rockside.io) and connect using your github account.
You will get your API KEY that will allow you to access Rockside API.

### Deploy your forwarder contract

To relay your transaction you need to deploy a forwarder contract.

This contract receive your transaction sent by Rockside, relay it to your Dapp contract and payback for the gas consumed using a gas price lower or equal to the specified Gas Price Limit.

To deploy a forwarder contract you have to specify an owner account.

By default forwarders only accept transactions coming from rockside signers. Only the owner of the forwarder can modify the list of authorized sender.

Execute this request:

```bash
curl --location --request POST 'https://api.rockside.io/ethereum/ropsten/forwarder' \
--header 'apikey: YOUR_API_KEY' \
--header 'Content-Type: application/json' \
--data-raw '{
	"owner": "0xf845b2501A69eF480aC577b99e96796c2B6AE88E"
}'
```

You get:

```
{
    "address": "0xa83E94cA4A9D92009C1Bf6dCA54b3E34D4463138",
    "transaction_hash": "0x6663a81a1a827c4bf2301eb169de900c51d2b6e4e2c26d503dce10888f8cdee9",
    "tracking_id": "01E9ZSDHMYYFMW3E1CVQ9ADVHK"
}
```

### Send ETH to your Forwarder contract

On ropsten, we fund your forwarder with 0.05 ETH when its deployed.
When your credit is consumed you have to fund it by sending ETH to your Forwarder contract address.

### Sign your relay message

Here is a example of signing a message to interact with a contract deployed by Rokcside.

Create a new node project.

```bash
npm init
```

Install Rockside Wallet SDK

```bash
npm install @rocksideio/rockside-wallet-sdk
```

On index.js add:

```js
const  Wallet = require('@rocksideio/rockside-wallet-sdk/lib/wallet.js')
const  Hash = require('@rocksideio/rockside-wallet-sdk/lib/hash.js')

const wallet = Wallet.BaseWallet.createRandom();

const domain = { chainId: 3, verifyingContract: '0xa8F87be466D1bDff91E6A8E44Be47bF767432638' };
const metatx = {
  relayer: "",
  signer: wallet.getAddress(),
  to: "",
  value: 0,
  data: [],
  nonce: 0,
  gasLimit: 0,
  gasPrice: 0
};

const hash = Hash.executeMessageHash(domain, metatx);
wallet.sign(hash).then((value) => {
  console.log("Signer: "+wallet.getAddress())
  console.log("Signature: "+value)
});
```

Run the script

```bash
npm index.js
```

You get:

```bash
Signer: 0xeaD23030fe26C5965FDf6D5C72C575689E2F51D2
Signature: 0xf68b22a18b28ec88751a270ba1575634134f09847ed36587fbc51d5d2de1aef927d8cec7d7d0f870c7fc5ecfd59e9407f5b2c0ce0824dc988de427aaede89f681c
```

### Use rockside to Relay your transaction.

Call rockside API to relay your transaction:

```bash
curl --location --request POST 'https://api.rockside.io/ethereum/ropsten/FORWARDER_ADDRESS/relay' \
--header 'apikey: YOUR_API_KEU' \
--header 'Content-Type: application/json' \
--data-raw '{
  "value": "0",
  "nonce": "0",
  "destination_contract": "0xa8F87be466D1bDff91E6A8E44Be47bF767432638"
  "signer": "YOUR_WALLET_ADDRESS",
  "signature": "YOUR_SIGNATURE"
}'
```

You get:

```json
{
    "transaction_hash": "0x2968698cb90ec9d95f8656d28bb29593029c79bcd22b42dc6b9469cb03729e2a",
    "tracking_id": "01EA266P6PKN0ZY01P0Q6G7WR5"
}
```

You can follow your transaction on etherscan or you can use Rockside API to keep track of your transaction even if it's replayed to bump the gas price

To follow your transaction using its tracking Id use:

```bash
curl --location --request GET 'https://api.rockside.io/ethereum/ropsten/transactions/TX_TRACKING_ID' \
--header 'apikey: YOUR_APIKEY' \
```

You Get:

```json
{
    "transaction_hash": "0x2968698cb90ec9d95f8656d28bb29593029c79bcd22b42dc6b9469cb03729e2a",
    "tracking_id": "01EA266P6PKN0ZY01P0Q6G7WR5",
    "customer_id": "4",
    "kind": "relay",
    "from": "0xdd0f36e17474e8cbf9c4e483d02a1cf34f41550a",
    "to": "0x7bb7703f2c601a54b484add52a07afad9c9f495e",
    "data_length": 484,
    "value": 0,
    "gas": 87601,
    "gas_price": 19270,
    "chain_id": 3,
    "price_mode": 1,
    "deadline": "2020-06-05T12:15:03.337Z",
    "receipt": {
        "status": 1,
        "cumulative_gas_used": 6332041,
        "logs": [],
        "transaction_hash": "0x2968698cb90ec9d95f8656d28bb29593029c79bcd22b42dc6b9469cb03729e2a",
        "contract_address": "0x0000000000000000000000000000000000000000",
        "gas_used": 86687,
        "block_hash": "0xd21a10cc344cb84faaab6725de8dedf51ae8deaacba62c6e0a570dc2578481f2",
        "block_number": 8035552,
        "transaction_index": 8
    },
    "created": "2020-06-05T12:10:03.337Z",
    "updated": "2020-06-05T12:11:23.632Z",
    "status": "success",
    "mining_time_estimate": 80
}
```

