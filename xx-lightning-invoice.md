# BOLT #xx: Unified Lightning Network Invoice

Defining structure and MIME type for Lightning Network Invoice containing BOLT-11 payload. This will help reduce size of data included in QR codes, maintain backwards compatibility and leverage existing `bitcoin:` URI scheme.

## Motivation

The primary motivation is unifying `bitcoin:` URI schemes for both on-chain and off-chain payments. Additional benefits are shorter URIs and less data to encode in QR codes. Standardization of MIME type and invoice structure will streamline implementation in Lightning Network wallets.

## Specification

A new MIME type  (Content-Type in HTML/email headers) shall be:

Message | Type/Subtype
--------|---------------
Lightning Network Invoice | application/bitcoin-lightning-invoice

`bitcoin:` url scheme is extended with optional `r=ENCODED_URL` as per [BIP-72](https://github.com/bitcoin/bips/blob/master/bip-0072.mediawiki). Compatible Lightning Network wallets would parse this query parameter and, if present, send a GET request to extracted URL with HTTP header `Accept: application/bitcoin-lightning-invoice`.

If response sent by web server returns HTTP code 200 and has HTTP header `Content-Type: application/bitcoin-lightning-invoice` the lightning wallet would proceed with decoding the data as described below.

![Bitcoin Wallet](xx-lightning-invoice/2.png)

The format of the server is formatted as example below:

<pre>
Content-Type: application/bitcoin-lightning-invoice

{
  "bolt11" : "lnbc20m1pvjluezhp58yjmdan79s6qqdhdzgynm4zwqd5d7xmw5fk98klysy043l2ahrqspp5qqqsyqcyq5rqwzqfqqqsyqcyq5rqwzqfqqqsyqcyq5rqwzqfqypqfppj3a24vwu6r8ejrss3axul8rxldph2q7z9kmrgvr7xlaqm47apw3d48zm203kzcq357a4ls9al2ea73r8jcceyjtya6fu5wzzpe50zrge6ulk4nvjcpxlekvmxl6qcs9j3tz0469gq5g658y",

  # Optional
  "nodeInformation" :
  {
    "nodeId" : "039ae2ef0c151e1e9032521002893dee94a5751c827e4941b5167f9d655a997c6f@lnnode.example.com",

    # Optional
    "alias" : "Example Store",

    # Optional (default: false)
    "prefundingAllowed": true
  },

  # Optional 
  "onchainPayment" :
  {
    "address" : "bc1....",
    "amount" : 102000
  }  
}
</pre>

This invoice would be parsed as:

 * `bolt11` as specified by [BOLT specification](https://github.com/lightningnetwork/lightning-rfc/blob/master/11-payment-encoding.md)
 * `nodeInformation` in case the wallet fails to route a Lightning payment, it can propose to open a channel to the user.
 * `nodeId` the information necessary to the wallet to create a new channel.
 * `alias` if the user creates a new channel, this is user friendly name wallet should use for the channel. 
 * `prefundingAllowed` if `true` and the payment failed to route on Lightning Network, the wallet can pay the invoice by prefunding the channel. (default: `false`)
 * `onchainPayment.address`, if the payment failed to route on Lightning Network, the wallet may propose to the user to pay on-chain (Potentially at the same time as opening a channel for future transactions)
 * `onchainPayment.amount`, the amount of satoshi to pay for settling the invoice on chain.
 
 If the object `onchainPayment` is present, the `f` field of `bolt11` must be completely ignored. 
 
 A payment done on-chain incurs future additional cost for the merchant who need to pay for transferring the received unspent transaction output.

 The merchant may choose to impact this cost to a customer choosing onchain payment method.  However, `BOLT11` does not allow different invoice amount for onchain payment made to the address in the `f` field. To overcome this limitation, a separate `onchainPayment` object is needed.

 ## Backward compatibility

If the user has a bitcoin wallet supporting only on-chain payment, the wallet would use [BIP70](https://github.com/bitcoin/bips/blob/master/bip-0070.mediawiki) or [BIP72](https://github.com/bitcoin/bips/blob/master/bip-0072.mediawiki), which are two protocols already widely deployed.

![Lightning Wallet](xx-lightning-invoice/1.png)
 
 ## References
 
  - [BOLT-11: Invoice Protocol for Lightning Payments](https://github.com/lightningnetwork/lightning-rfc/blob/master/11-payment-encoding.md)
  - [BIP72: bitcoin: uri extensions for Payment Protocol](https://github.com/bitcoin/bips/blob/master/bip-0072.mediawiki)
  - [BIP70: Payment Protocol](https://github.com/bitcoin/bips/blob/master/bip-0070.mediawiki)
  
 ## Authors
 
  - Predrag Tomasevic &lt;codeoverwhelming@outlook.com&gt;
  - Nicolas Dorier &lt;nicolas.dorier@gmail.com&gt;
 
