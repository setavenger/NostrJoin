# NostrJoin (Payjoin v2)
A nostr based Payjoin upgrade that does not require the sender or receiver to run their own server

## Abstract
This proposal suggests a new process for exchanging the PSBTs between the two parties participating in a Payjoin.
The fundamental protocol stays unchanged and can be found [here](https://github.com/bitcoin/bips/blob/master/bip-0078.mediawiki).
Using the novel network nostr would allow participants to utilize a decentralized communication platform to coordinate their Payjoin. 
Utilizing nostr DMs an end-to-end-encrypted communication protocol between to parties we can create a workflow which is secure and blends into already existing DMs.   

## Overview

### Current workflow

Below we can see the communication between the receiver and the sender as defined by BIP-78.
BIP-78 defines the workflow as follows:
- Receiver provides the sender with a BIP-21 URI which contains a payjoin server endpoint
- The sender then makes a http request to the payjoin server of the receiver with an `Original PSBT` in the body
- The receiver's Payjoin server gives a http response with the payjoin proposal PSBT
- The sender verifies the `Payjoin Proposal PSBT` and broadcasts the `Payjoin Transaction` to the Bitcoin Network

This process requires that Both parties stay online during the entire process.
Furthermore, it requires the Receiver to run a payjoin server in order to communicate the `Original PSBT` and the `Payjoin Proposal PSBT`. 

```text
+----------+                        +--------+         +-----------------+
| Receiver |                        | Sender |         | Bitcoin Network |
+----+-----+                        +---+----+         +-------+---------+
     |       +-----------------+        |                      |
     +-------+ BIP21 with ?pj= +------->+                      |
     |       +-----------------+        |                      |
     |                                  |                      |
     |        +---------------+         |                      |
     +<-------+ Original PSBT +---------+                      |
     |        +---------------+         |                      |
     |                                  |                      |
     |       +------------------+       |                      |
     |       | Payjoin Proposal |       |                      |
     +-------+      PSBT        +------>+                      |
     |       +------------------+       |                      |
     |                                  |   +--------------+   |
     |                                  |---+ Payjoin      |   |
     |                                  |   | Transaction  +-->+
     |                                  |   +--------------+   |
     +                                  +                      +
```

### Nostr
By replacing the http requests/responses with nostr DMs we can modify this workflow to be asynchronous and relieve both of the parties of hosting their own server.
The decentralized nature of nostr allows the receiver to either host their own relay or rely on the existing relays which are around.
The receiver can suggest several relays that she will listen to for an incoming DM with the `Original PSBT`.
As the Relays store the notes for some time (Relays usually store notes for a decent amount of time and to make sure DMs find their way several relays can be chosen) the Receiver can check for nostr DMs when she comes back online. 
Then she can answer the nostr DMs with the `Payjoin Proposal PSBT`s.  

It is recommended that sender and receiver create a new keypair for each transaction in order to break any possible heuristics.

## Specification
The majority of the existing process stays the same, the only changes are: 
- This proposal adds to parameters `pjnpub=` and `pjnostrrelays=`
- The interaction/communication rails between the parties.

The remaining parts of the protocol stay the same.

### Parameters

This proposal defines two new URI query parameters `pjnpub=` and `pjnostrrelays=`.
`pjnpub=` defines the npub to which the DM should be sent and `pjnostrrelays=` tells the sender which relays he should include to maximize the chances of the receiver seeing the DM.

A possible BIP-21 URI string could look like this 
`bitcoin:175tWpb8K1S7NmH4Zx6rewF9WQrcZv245W?pjnpub=npub1jfpffumqvyfuj7xgtmk3pe3npultvn2feh7n2qn5657xrsx6fzkq9hstew&pjnostrrelays=wss://nostr-pub.wellorder.net,wss://relay.damus.io,wss://relay.nostr.info`.

### Communication

This basically tells the sender that he should create the `Original PSBT` sending to `175tWpb8K1S7NmH4Zx6rewF9WQrcZv245W` and the `Original PSBT` should be sent via nostr DM to this `npub1jfpffumqvyfuj7xgtmk3pe3npultvn2feh7n2qn5657xrsx6fzkq9hstew` npub.
Among the relays to which the sender broadcasts the DM should be the following relays `wss://nostr-pub.wellorder.net`, `wss://relay.damus.io` and `wss://relay.nostr.info`.
The sender should try to send to all the listed relays in order to maximize the chance of the receiver seeing the DM.

- After receiving the DM with `Original PSBT` the receiver does the process as defined in BIP-78 and answers the DM with the `Payjoin Proposal PSBT`.
- When receiving the DM with the `Payjoin Proposal PSBT` the sender verifies the psbt and broadcasts the `Payjoin Transaction`.

This workflow allows for asynchronous interaction between the two parties.
As part of the asynchronous workflow it is optional to broadcast the `Original Psbt` after a timeout. 
Wallet developers should find a good middle ground between waiting and broadcasting the `Original Psbt` depending on the urgency of the transaction.    

## Wallet Developers
Wallet Developers should make sure to store the generated nostr key pairs in order to know which npubs have to be checked for DMs.


## Implementations
[Javascript Receiver (WIP) and Sender](https://github.com/setavenger/payjoin-client)