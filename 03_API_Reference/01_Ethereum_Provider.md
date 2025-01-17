# API Reference

MetaMask injects a global API into websites visited by its users at `window.ethereum` (Also available at `window.web3.currentProvider` for legacy reasons). This API allows websites to request user login, load data from blockchains the user has a connection to, and suggest the user sign messages and transactions. You can use this API to detect the user of a web3 browser.

```javascript
if (typeof window.ethereum !== 'undefined'
|| (typeof window.web3 !== 'undefined')) {

  // Web3 browser user detected. You can now use the provider.
  const provider = window['ethereum'] || window.web3.currentProvider
}
```

The provider API itself is very simple, and wraps [Ethereum JSON-RPC](./JSON_RPC_API) formatted messages, which is why developers usually use a convenience library for interacting with the provider, like [web3](https://www.npmjs.com/package/web3), [truffle](https://truffleframework.com/), [ethjs](https://www.npmjs.com/package/ethjs), [Embark](https://embark.status.im/), or others. From those tools, you can generally find sufficient documentation to interact with the provider, without reading this lower-level API.

However, for developers of convenience libraries, and for developers who would like to use features that are not yet supported by their favorite libraries, knowledge of the provider API is essential.

Some simplifications are coming soon to this API, which you can [read more about here](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1193.md).

## Properties

These properties can be used to check the current state of the connected user, which can be important things to verify before sending a transaction:

### ethereum.networkVersion

Returns a numeric string representing the current blockchain's network ID. A few example values:

'1': Ethereum Main Network
'2': Morden Test network
'3': Ropsten Test Network
'4': Rinkeby Test Network
'42': Kovan Test Network

### ethereum.selectedAddress

Returns a hex-prefixed string representing the current user's selected address, ex: `"0xfdea65c8e26263f6d9a1b5de9555d2931a33b825"`.

### ethereum.isMetaMask

Returns `true` or `false`, representing whether the user has MetaMask installed.

## Methods

### ethereum.enable()

Requests the user provides an ethereum address to be identified by. Returns a promise of an array of hex-prefixed ethereum address strings.

Example usage (ES6), assuming [async function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function):

```javascript
try {
  const accounts = await ethereum.enable()
  // You now have an array of accounts!
  // Currently only ever one:
  // ['0xFDEa65C8e26263F6d9A1B5de9555D2931A33b825']

} catch (error) {
  // Handle error. Likely the user rejected the login
  console.error(error)
}
```

Example usage (ES5):

```javascript
ethereum.enable()
.then(function (accounts) {
  // You now have an array of accounts!
  // Currently only ever one:
  // ['0xFDEa65C8e26263F6d9A1B5de9555D2931A33b825']
})
.catch(function (error) {
  // Handle error. Likely the user rejected the login
  console.error(error)
})
```

### ethereum.send(options)

The primary recommended method to send a message to the web3 browser. Message format maps to the format of [the Ethereum JSON-RPC API](https://github.com/ethereum/wiki/wiki/JSON-RPC#json-rpc-methods).

Returns a promise in the form of a JSON-RPC response.

Here's an example of everyone's favorite method, sending a transaction, which is both how Ether is sent, and how smart contract methods are called:
```javascript
params: [{
  "from": "0xb60e8dd61c5d32be8058bb8eb970870f07233155",
  "to": "0xd46e8dd67c5d32be8058bb8eb970870f07244567",
  "gas": "0x76c0", // 30400
  "gasPrice": "0x9184e72a000", // 10000000000000
  "value": "0x9184e72a", // 2441406250
  "data": "0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f072445675"
}]

ethereum.send({
  method: 'eth_sendTransaction',
  params: params,
  from: accounts[0], // Provide the user's account to use.
})
.then(function (result) {
  // The result varies by method, per the JSON RPC API.
  // For example, this method will return a transaction hash on success.
})
.catch(function (error) {
 // Like a typical promise, returns an error on rejection.
})
```
### ethereum.sendAsync(options, callback)

_To be superceded by the promise-returning send() method in [EIP 1193](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1193.md)._

Sends a message to the web3 browser. Message format maps to the format of [the Ethereum JSON-RPC API](https://github.com/ethereum/wiki/wiki/JSON-RPC#json-rpc-methods).

Here's an example of everyone's favorite method, sending a transaction, which is both how Ether is sent, and how smart contract methods are called:
```javascript
params: [{
  "from": "0xb60e8dd61c5d32be8058bb8eb970870f07233155",
  "to": "0xd46e8dd67c5d32be8058bb8eb970870f07244567",
  "gas": "0x76c0", // 30400
  "gasPrice": "0x9184e72a000", // 10000000000000
  "value": "0x9184e72a", // 2441406250
  "data": "0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f072445675"
}]

ethereum.sendAsync({
  method: 'eth_sendTransaction',
  params: params,
  from: accounts[0], // Provide the user's account to use.
}, function (err, result) {
  // A typical node-style, error-first callback.
  // The result varies by method, per the JSON RPC API.
  // For example, this method will return a transaction hash on success.
})
```

### ethereum.autoRefreshOnNetworkChange

When the network is changed, MetaMask will reload any pages that have made requests to the provider. This automatic reload behavior will be removed in a future release of MetaMask, but in the meantime it can be disabled with this flag.

To disable auto-refresh on a network change you can do:

```javascript
ethereum.autoRefreshOnNetworkChange = false;
```

This can be toggled on or off at any time.

Note: Setting this flag to `true` results in the default behavior, which is subject to change. If your site relies upon MetaMask reloading it upon network change, you will need to trigger the reload yourself in a `networkChanged` event handler to ensure it continues to work with future releases.

### ethereum.on(eventName, callback)

The provider supports listening for some events:
- `accountsChanged`, returns updated account array.
- `networkChanged`, returns network ID string.

Example
```javascript
ethereum.on('accountsChanged', function (accounts) {
  // Time to reload your interface with accounts[0]!
})
```

Note: At the moment, the `networkChanged` event is only useful if you disable auto-refresh on network change by setting [`ethereum.autoRefreshOnNetworkChange`](#ethereum.autorefreshonnetworkchange) to `false`. Otherwise, MetaMask will default to auto-reloading pages upon network change if they have made requests to the provider. This default will be changing in a future release.
