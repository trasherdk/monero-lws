# monero-lws ZeroMQ Usage
Monero-lws uses ZeroMQ-RPC to retrieve information from a Monero daemon,
ZeroMQ-SUB to get immediate notifications of blocks and transactions from a
Monero daemon, and ZeroMQ-PUB to notify external applications of payment_id
and new account (web)hooks.

## External "pub" socket
The bind location of the ZMQ-PUB socket is specified with the `--zmq-pub`
option. Users are still required to "subscribe" to topics:
  * `json-full-payment_hook`: A JSON object of a single webhook payment event
    that has recently triggered (identical output as webhook).
  * `msgpack-full-payment_hook`: A msgpack object of a webhook payment events
    that have recently triggered (identical output as webhook).
  * `json-full-new_account_hook`: A JSON object of a single new account
    creation that has recently triggered (identical output as webhook).
  * `msgpack-full-new_account_hook`: A msgpack object of a single new account
    creation that has recently triggered (identical output as webhook).
  * `json-minimal-scanned`: A JSON object of a list of user primary addresses,
    with their new height and block hash.
  * `msgpack-minimal-scanned:` A msgpack object of a list of user primary
    addresses with their new height and block hash.
  * `json-full-spend_hook': A JSON object of a webhook spend event that has
    recently triggerd (identical output as webhook).
  * `msgpack-full-spend_hook`: A msgpack object of a single new account
    creation that has recently triggered (identical output as webhook).


### `json-full-payment_hook`/`msgpack-full-payment_hook`
These topics receive PUB messages when a webhook ([`webhook_add`](administration.md)),
event is triggered for a payment (`tx-confirmation`). If the specified URL is
`zmq`, then notifications are only done over the ZMQ-PUB socket, otherwise the
notification is sent over ZMQ-PUB socket AND the specified URL. Invoking
`webhook_add` with a `payment_id` of zeroes (the field is optional in
`webhook_add`), will match on all transactions that lack a `payment_id`.

Example of the "raw" output from ZMQ-SUB side:

```json
json-full-payment_hook:{
  "index": 2,
  "event": {
    "event": "tx-confirmation",
    "payment_id": "4f695d197f2a3c54",
    "token": "single zmq wallet",
    "confirmations": 1,
    "event_id": "3894f98f5dd54af5857e4f8a961a4e57",
    "tx_info": {
      "id": {
        "high": 0,
        "low": 5666768
      },
      "block": 2265961,
      "index": 1,
      "amount": 3117324236131,
      "timestamp": 1687301600,
      "tx_hash": "ef3187775584351cc5109de124b877bcc530fb3fdbf77895329dd447902cc566",
      "tx_prefix_hash": "064884b8a8f903edcfebab830707ed44b633438b47c95a83320f4438b1b28626",
      "tx_public": "54dce1a6eebafa2fdedcea5e373ef9de1c3d2737ae9f809e80958d1ba4590d74",
      "rct_mask": "4cdc4c4e340aacb4741ba20f9b0b859242ecdad2fcc251f71d81123a47db3400",
      "payment_id": "4f695d197f2a3c54",
      "unlock_time": 0,
      "mixin_count": 15,
      "coinbase": false
    }
  }
}

```

Notice the `json-full-payment_hook:` prefix - this is required for the ZMQ PUB/SUB
subscription model. The subscriber requests data from a certain "topic" where
matching is done by string prefixes.

> `index` is a counter used to detect dropped messages.

> The `block` and `id` fields in the above example are NOT present when
`confirmations == 0`.

### `json-full-new_account_hook`/`msgpack-full-new_account_hook`
These topics receive PUB messages when a webhook ([`webhook_add`](administration.md)),
event is triggered for a new account (`new-account`). If the specified URL is
`zmq`, then notifications are only done over the ZMQ-PUB socket, otherwise the
notification is sent over ZMQ-PUB socket AND the specified URL. Invoking
`webhook_add` with a `payment_id` of zeroes (the field is optional in
`webhook_add`), will match on all transactions that lack a `payment_id`.

Example of the "raw" output from ZMQ-SUB side:

```json
json-full-new_account_hook:{
  "index": 2,
  "event": {
    "event": "new-account",
    "event_id": "c5a735e71b1e4f0a8bfaeff661d0b38a",
    "token": "",
    "address": "9zGwnfWRMTF9nFVW9DNKp46aJ43CRtQBWNFvPqFVSN3RUKHuc37u2RDi2GXGp1wRdSRo5juS828FqgyxkumDaE4s9qyyi9B"
  }
}
```

Notice the `json-full-new_account_hook:` prefix - this is required for the ZMQ
PUB/SUB subscription model. The subscriber requests data from a certain "topic"
where matching is done by string prefixes.

> `index` is a counter used to detect dropped messages.

### `json-minimal-scanned`/`msgpack-minimal-scanned`
These topics receive PUB messages when a thread has finished scanning 1+
accounts. The last block height and hash is sent.

Example of the "raw" output from ZMQ-SUB side:
```json
json-minimal-scanned:{
  "index": 13,
  "event": {
    "height": 2438536,
    "id": "9197e1c6f3de28a98dfc579325903e5416ef1ba2681043c54b5fff0d39645a7f",
    "addresses": [
      "9xkhhJSa7ZhS5sAcTix6ozL14RwdgxbV7JZVFW4rCghN7GidutaykfxDHfgW45UPiCTXncuvZ91GNSGgxs3b2Cin9TU8nP3"
    ]

> `index` is a counter used to detect dropped messages.

### `json-full-spend_hook`/`msgpack-full-spend_hook`
These topics receive PUB messages when a webhook ([`webhook_add`](administration.md)),
event is triggered for a spend (`tx-spend`). If the specified URL is
`zmq`, then notifications are only done over the ZMQ-PUB socket, otherwise the
notification is sent over ZMQ-PUB socket AND the specified URL. Invoking
`webhook_add` with a `payment_id` or `confirmation` results in a NOP because
both fields are unused for spends. This event is only triggered on
confirmation==1 (`confirmation` field on `webhook_add`s have no effect, and
mempool spends are not scanned). The intent is to notify the user of unexpected
spend operations. The end user will need to use `tx_info.input.image`,
`tx_info.source.index`, and `tx_info.source.tx_public` to determine if the
output was actually spent or being used as a decoy.

Example of the "raw" output from ZMQ-SUB side:

```json
json-full-spend_hook:{
  "index": 0,
  "event": {
    "event": "tx-spend",
    "token": "spend-xmr",
    "event_id": "7ff047aa74e14f4aa978469bc0eec8ec",
    "tx_info": {
      "input": {
        "height": 2464207,
        "tx_hash": "97d4e66c4968b16fec7662adc9f8562c49108d3c5e7030c4d6dd32d97fb62540",
        "image": "b0fe7acd9e17bb8b9ac2daae36d4cb607ac60ed8a101cc9b2e1f74016cf80b24",
        "source": {
          "high": 0,
          "low": 6246316
        },
        "timestamp": 1711902214,
        "unlock_time": 0,
        "mixin_count": 15,
        "sender": {
          "maj_i": 0,
          "min_i": 0
        }
      },
      "source": {
        "id": {
          "high": 0,
          "low": 6246316
        },
        "amount": 10000000000,
        "mixin": 15,
        "index": 0,
        "tx_public": "426ccd6d39535a1ee8636d14978581e580fcea35c8d3843ceb32eb688a0197f7"
      }
    }
  }
}
```

> `index` is a counter used to detect dropped messages
