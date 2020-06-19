# Wallet Server API

The wallet exposes a JSON-RPC API over Websocket by default at the URL `127.0.0.1:21338`.
It can be set in the Witnet configuration file as follows:

```toml
[wallet]
node_url = "127.0.0.1:21338"
```


## Summary

| Method Name                                 | Request Params                                   | Response                                      |
| ------------------------------------------- | ------------------------------------------------ | --------------------------------------------- |
| [get_wallet_infos](#get_wallet_infos)       | (none)                                           | `wallet_info[]`                               |
| [create_mnemonics](#create_mnemonics)       | length                                           | `mnemonics`                                   |
| [validate_mnemonics](#validate_mnemonics)   | seed_source, seed_data                           | `valid`                                       |
| [create_wallet](#create_wallet)             | name, caption, seed_source, seed_data, password  | `wallet_id`                                   |
| [unlock_wallet](#unlock_wallet)             | wallet_id, password                              | `session_id`, `session_expiration_secs`, ...  |
| [update_wallet](#update_wallet)             | `session_id`, `wallet_id`, `name`, `caption`     | `success`                                     |
| [lock_wallet](#update_wallet)               | `session_id`, `wallet_id`                        | `success`                                     |
| [close_session](#close_session)             | `session_id`                                     | `success`                                     |



## Wallet API Endpoints:


### get_wallet_infos
    
The JsonRPC method `get_wallet_infos` returns a list of all wallets with names and captions stored on the wallet server database.

Request (no parameters):

```json
{
  "jsonrpc": "2.0",
  "method": "get_wallet_infos",
  "id": "1"
}
```

Response with an array of all wallets stored inn the server with their information, `infos[]`, containing:

- `id`: *String*, ID associated with the given wallet. 
- `name`: *String*, human-friendly name for the given wallet.
- `caption`: *String*, human-friendly caption for the given wallet.

```json
{
  "jsonrpc": "2.0",
  "result": 
  {
    "infos": 
    [
      {
        "caption":null, 
        "id":"81ccbf4548cfeba37cef93dc64e7f0d8fb410e3967bb40160a36aa362943c520",
        "name":null
      },
      {
        "caption":"caption text",
        "id":"9fa1d779afea88a29768dd05647e37b2f64fc103c1081b0ee9e62fb283f5cd02",
        "name":"wallet name"
      }		
    ]
  },
  "id": 1
}
```


### create_mnemonics

The JsonRPC method `create_mnemonics` is used to generate a [BIP39 mnemonic sentence](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) that can be used to generate a new [HD wallet](https://en.bitcoinwiki.org/wiki/Deterministic_wallet).

Request with parameters:

- `length`: *integer* indicating how many words the mnemonic sentence should have. Must be one of these: `12`, `15`, `18`, `21` or `24`.

```json
{
  "jsonrpc": "2.0",
  "method": "create_mnemonics",
  "params": { 
    "length": 12 
  },
  "id": 1
}
```

Response:

 - `mnemonics`: *String*, list of words of the mnemonic sentences.

```json
{
  "jsonrpc": "2.0",
  "result": {
    "mnemonics": "day voice lake monkey suit bread occur own cattle visit object ordinary"
  },
  "id": 1
}
```


### validate_mnemonics

The JsonRPC method `validate_mnemonics` is used to validate that validity of the seed source that might be used to generate a new wallet.

Request with parameters:

- `seed_source`: *`"mnemonics" | "xprv"`* literal to identify if the seed source is of the type *mnemonics* or *xprv*.
- `seed_data`: *String* containing the used seed data, either a list of mnemonic words or a `xprv`.

```json
{
  "jsonrpc": "2.0",
  "method": "validate_mnemonics",
  "params": { 
  	"seed_source": "mnemonics",
    "seed_data": "day voice lake monkey suit bread occur own cattle visit object ordinary"
  },
  "id": 1
}
```

Response:

- `valid`: *Boolean*, true if valid seed in form of *mnemonics* or *xpriv*.

```json
{
  "jsonrpc": "2.0",
  "result": {
    "valid": true
  },
  "id": 1
}
```


### create_wallet

The JsonRPC method `create_wallet` is used to generate a new Master Key for an empty [HD wallet](https://en.bitcoinwiki.org/wiki/Deterministic_wallet) that is stored encrypted in the file system.

Request with parameters:

- `name` (optional): *String*, human-friendly name for your the wallet.
- `caption` (optional): *String*, human-friendly caption for your the wallet.
- `seed_source`: *`"mnemonics" | "xprv"`* literal to identify if the seed source is of the type *mnemonics* or *xprv* and determines how the HD wallet master key will be generated from the data sent in the `seedData` param.
- `seed_data`: *String*, data used for generating the new HD wallet master key.
- `password`: *String*, password that will seed the key used to encrypt the wallet in the file system. The password must have at least eight characters.

```json
{
  "jsonrpc": "2.0",
  "method": "create_wallet",
  "params": {
    "name": "Wallet #1",
    "caption": "Personal use",
    "seed_source": "mnemonics",
    "seed_data": "exotic demand way fatigue skull poverty happy divide scrub seed jeans novel",
    "password": "12345678"
  },
  "id": 1
}
```

Response:

- `wallet_id`: *String*, ID associated with the given wallet.

```json
{
  "jsonrpc": "2.0",
  "result": {
    "wallet_id": "6c344625884c2f910065ab170dc18ad3cbbc03c7234507c7c22dbd78e3b26667"
  },
  "id": 1
}
```


### unlock_wallet

The JsonRPC method `unlock_wallet` is used to *unlock* the wallet with the specified identifier by providing a decryption key. This key will be hold in memory until the wallet is locked again. As long as a wallet is unlocked, you can operate it without having to supply the password again by just using the session id until it expires.

Request with parameters:

- `wallet_id`: *String*, the ID associated to the wallet. See [get_wallet_infos](#get-wallet-infos).
- `password`: *String*, the password that unlocks the wallet.

```json
{
  "jsonrpc": "2.0",
  "method": "unlock_wallet",
  "params": {
    "wallet_id": "6c344625884c2f910065ab170dc18ad3cbbc03c7234507c7c22dbd78e3b26667",
    "password": "12345678"
  },
  "id": 1
}
```

Response:

- `session_id`: *number*, generated identifier for the current wallet session.
- `session_expiration_secs`: *number*, amount of seconds after which the session will expire.
- `account_balance`: *number*, wallet's account balance in nano Wits.
- `name`: *String*, human-friendly name for your the wallet.
- `caption`: *String*, human-friendly caption for your the wallet.
- `current_account`: *number*, identifies the current active account in the session (the current version only supports the default account `0`).
- `available_accounts`: *Array<number>*, list of available accounts in the wallet.

```json
{
  "jsonrpc": "2.0",
  "result": {
    "account_balance": 0,
    "available_accounts": [
      0
    ],
    "caption": null,
    "current_account": 0,
    "name": null,
    "session_expiration_secs": 3200,
    "session_id": "9c4f690a50de45b91bb4a5d7fc964c6853ca4eb29fa4ed3e2c9ddfd3e2da45e7"
  },
  "id": 1
}
```


### update_wallet

The JsonRPC method `update_wallet` is used to update the name and/or caption of an existing wallet.

Request with parameters:

- `wallet_id`: *String*, the ID associated to the wallet. See [get_wallet_infos](#get-wallet-infos).
- `session_id`: *number*, generated identifier obtained from unlocking the wallet. See [Unlock Wallet](#unlock_wallet).
- `name`: *String*, human-friendly name for your the wallet.
- `caption`: *String*, human-friendly caption for your the wallet.

```json
{
  "jsonrpc": "2.0",
  "method": "update_wallet",
  "params": {
    "session_id": "f1188c907e581f067ac589cf962c7f4fea9443e93d8df10a945e7d17fae49870",
    "wallet_id": "6c344625884c2f910065ab170dc18ad3cbbc03c7234507c7c22dbd78e3b26667",
    "name": "New Name",
    "caption": "New Caption"
  },
  "id": 1
}
```

Response:

- `success`: *Boolean*, reporting if the wallet's update was successfull.

```json
{
  "jsonrpc": "2.0",
  "result": {
    "success": true
  },
  "id": 1
}
```


### lock_wallet

The JsonRPC method `lock_wallet` is used to *lock* the wallet with the specified id and close the active session. The decryption key for that wallet (hold in memory) is forgotten and the wallet server will be unable to update that wallet information until it is unlocked again.

Request with parameters:

- `session_id`: *String*, session ID assigned to you when you unlocked the wallet. See [unlock_wallet](#unlock_wallet).
- `wallet_id`: *String*, ID associated to the wallet. See [get_wallet_infos](#get-wallet-infos).

```json
{
	"jsonrpc": "2.0",
  	"method": "lock_wallet",
  	"params": {
    	"session_id": "f1188c907e581f067ac589cf962c7f4fea9443e93d8df10a945e7d17fae49870",
    	"wallet_id": "6c344625884c2f910065ab170dc18ad3cbbc03c7234507c7c22dbd78e3b26667"
  	},
  	"id": "1"
}
```

Response:

- `success`: *Boolean*, reporting if the wallet was successfully locked.

```json
{
	"jsonrpc": "2.0",
  	"result": {
      "success": true
    },
  	"id": "1"
}
```


### close_session

The JsonRPC method `close_session` is used to close an active session without locking the currently unlocked wallet.

Request with parameters:

- `session_id`: *String*, session ID assigned to you when you unlocked the wallet. See [unlock_wallet](#unlock_wallet).

```json
{
	"jsonrpc": "2.0",
	"method": "close_session",
  	"params": {
    	"session_id": "9fa1d779afea88a29768dd05647e37b2f64fc103c1081b0ee9e62fb283f5cd02"
  	},
  	"id": "1"
}
```

Response:

- `success`: *Boolean*, reporting if the wallet was successfully locked.

```json
{
	"jsonrpc": "2.0",
  	"result": {
      "success": true
    },
  	"id": "1"
}
```
