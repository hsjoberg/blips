```
bLIP: XXX
Title: LN P2P Protocol for forwarding LNURL-pay requests
Status: Active
Author:  Hampus Sjöberg <hampus.sjoberg@protonmail.com>
Created: 2023-08-16
License: CC0
```

## Abstract

This bLIP describes a way to forward LNURL-pay requests from an LNURL-server
("Server") to a Lightning node ("Client") using Lightning P2P messaging.

The LNURL-pay JSON requests and responses SHOULD be forwarded verbatim
(with exception of the `callback` property) and the client is expected to return
valid LNURL-pay responses back.

In the case where the recipient client is offline, the server MAY return an
LNURL error message back to the Requester.

```
[Requester] <--HTTP--> [LNURL-pay Server] <--LN P2P--> [Client]
```

## Copyright

This bLIP is licensed under the CC0 license.

## Specification

Communication between the Client's Lightning node and the Server's Lightning
node is happening on the LN P2P code `33459`.

Upon receiving an incoming LNURL-pay request, the Server MUST send a JSON
stringified message to the Client. There are two requests and two responses as
per the LNURL-pay protocol.

Refer to LNURL-pay [LUD-06] and its extensions for specifics regarding each
request and response.

```Typescript
interface ILnurlPayForwardP2PMessage {
  id: number;
  request:
    | "LNURLPAY_REQUEST1"
    | "LNURLPAY_REQUEST1_RESPONSE"
    | "LNURLPAY_REQUEST2"
    | "LNURLPAY_REQUEST2_RESPONSE";
  data: any; // Data/JSON as defined in LUD-06 and other related LUDs
  metadata: {
    [key: string]: any;
  };
}
```

In order to keep track of requests and their corresponding responses, Server
MUST assign a unique `id` for each Request and Reponse bundle that the Client
MUST adhere to.

Server and Client MAY add additional metadata in the `metadata` prop.

### `LNURLPAY_REQUEST1`

This is the incoming LNURL-pay request and thus doesn't contain any data for the
Client.

```Javascript
{
  id: 1,
  request: "LNURLPAY_REQUEST1",
  data: {},
  metadata: {}
}
```

### `LNURLPAY_REQUEST1_RESPONSE`

This is initial response providing payment information.

```Javascript
{
  id: 1,
  request: "LNURLPAY_REQUEST1_RESPONSE",
  data: { // From LUD-06 
    tag: "payRequest",
    minSendable: 1000,
    maxSendable: 10000,
    metadata: JSON.stringify([
      ["text/plain", "Cheers!"],
    ]),
    callback: "" // MAY be ommitted as the Client may not know which endpoint Server will meet the Requester on
  },
  metadata: {}
}
```

#### The callback property

The Client might not know on what `callback` URL the LNURL-pay server is
supposed to meet the Requester on. Thus the Server MAY alter
`LNURLPAY_REQUEST1_RESPONSE` in order to provide the correct `callback` URL.

### `LNURLPAY_REQUEST2`:

This is the final request from the Requester.
Requester will make a GET request Server via the callback URL.

```Javascript
{
  id: 2,
  request: "LNURLPAY_REQUEST2",
  data: { // From LUD-06
    amount: 1000
  },
  metadata: {}
}
```

### `LNURLPAY_RESPONSE2`:

This is the final response and will contain the BOLT11 code for the Requester.

```Javascript
{
  id: 2,
  request: "LNURLPAY_REQUEST2_RESPONSE",
  data: { // From LUD-06
    pr: "lnbc1abcdef",
    routes: []
  },
  metadata: {}
}
```

### Additional metadata

By how `LNURLPAY_REQUEST1` is designed, the Client may not know which LNURL-pay
code or Lightning Address the incoming request is regarding.
The Server MAY provide additional metadata in the request:

```Javascript
{
  id: 1,
  request: "LNURLPAY_REQUEST1",
  data: {},
  metadata: {
    lightningAddress: "satoshi@lightning.com" // and/or
    lnurlPay: "LNURL123" // and/or
    lud17: "lnurlp:https://domain.com/pay"
  }
}
```

### Client unresponsive

In the case Client does not respond within a reasonable time, the Server SHUOLD
stop waiting and terminate the request.

<!-- ## Motivation -->


<!-- ## Backwards Compatibility -->


## Reference Implementations

### Server

* Lightning Box: https://github.com/hsjoberg/lightning-box/commit/6e94da34f2bac13f39b8c4cd199da90e9dc05d55

### Client

* Blixt Wallet: https://github.com/hsjoberg/blixt-wallet/pull/1130/commits/18df00821ccb70e3f6fb400af90240346e104135#diff-6531b1ce191e634230e7f21b87353ba425759cc556885be11fab8cd998f9a997

[LUD-06]: https://github.com/lnurl/luds/blob/luds/06.md
