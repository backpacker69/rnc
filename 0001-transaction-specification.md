# Transaction specification

- Status: unfinished
- Type: new feature
- Related components: /
- Start Date: 06-10-2016
- Discussion: (fill me in with link to RNC discussion - shepherd will complete this)

## Summary

Two basic types of PeerAsset transactions are specified.
- Deck spawn transaction
- Card transfer transaction
With the card issue and card burn transactions being special types of the card transfer transaction.

The deck spawn transaction registers a new asset to be traded. The owner of `vin[0]` is the only enitity that is allowed to transfer assets with a negative balance, resulting in a card issue transaction. The deckspawn transaction decides how much an asset can be divided (number_of_decimals) and whether it can get issue multiple times or not (issue_once).

The card transfer transaction transfers ownership of assets from one holder to the next. This works on a first come first serve basis following the serialization order on the blockchain. Once the balance of an account becomes too lower that a transfer transaction it issued, that transfer transaction is considered invalid. Topping up the balance of an account to make that transfer valid, requires that transaction to be resent so it gets serialized in the chain after the incoming transaction on that account.

The card issue transaction is a special case of the card transfer transaction originating from the owner of the deck spawn transaction. The owner is the only account that is allowed to hold a negative balance. This balance is used as a checksum to validate a correct computation of all account balances. If the deck spawn transaction specifies issue_once to be true, only the first serialized card issue transaction is considered valid.

The card burn transaction is a special case of the card transfer transaction sent to the owner of the deck spawn transaction. This results in the owner's balance to decrease in absolute value, meaning that the total amount of issued assets decreases. This transaction can be used as Proof of Burn.

## Motivation

This RNC is meant to iterate and agree on the PeerAsset transaction protocol.

## Detailed design

A PeerAssets transaction encodes it's information in the transaction's inputs (vin[]), outputs (vout[]) and a special meta-data output (OP_RETURN).

### Deck spawn transaction layout

For the deck spawn transaction, the following in and properties are specified:
* txnid: The unique identifier for this asset.
* vin[0]: The owner of the Asset. Ownership of the asset is proven by proving ownership over the Public Key Hash (PKH) or Script Hash (SH) originating vin[0].
* vout[0]: PeerAssets tag using a P2TH output. This tag registers the assets as a PeerAsset so that it can be discovered by PeerAsset clients. The minimum output amount is 0.005PPC, see P2TH section for for reasoning.
* vout[1] (OP_RETURN): Asset meta-data. A protobuf3 encoded message containing meta-data about the asset (ref. peerassets.proto).
* all other in and outputs are free to be used in any way. vout[2] will typically be used as a change output.

### Card transfer transaction layout

TODO

### P2TH tags for testnet and mainnet

The PeerAssets protocol requires a tag output to have a minimum value of 0.005PPC, half the peercoin transaction fee, to be considered valid. This has two reasons. It is a counter measure to tag spam. And it incentivizes UTXO cleanup. The minimum tag fee can be recovered from a previous P2TH UTXO so it doesn't pollute the node's UTXO tables. Having the minimum tag fee lower than the transaction fee ensures that this output is merged with other outputs to be spent, resulting in a smaller UTXO table on the peercoin nodes.

#### Deck spawn tags

The deck spawn tag private keys are publicly known so it can be imported in every ppcoin node to easily query for deck spawn transactions without the need for a block explorer and to allow any node to claim the tag fees resulting in an UTXO cleanup.

PPC mainnet:
- PAprod: PAprodpH5y2YuJFHFCXWRuVzZNr7Tw78sV - 7A6cFXZSZnNUzutCMcuE1hyqDPtysH2LrSA9i5sqP2BPCLrAvZM
- PAtest: PAtestVJ4usB4JQwZEhFrYRgnhKh8xRoRd - 79nanGVB5H5cGrpqN69F3v4rjyhXy5DiqF499TB5poF627Z1Gw4
PPC testnet:
- PAprod: miYNy9BbMkQ8Y5VaRDor4mgH5b3FEzVySr - 92NRcL14QbFBREH8runJAq3Q1viQiHoqTmivE8SNRGJ2Y1U6G3a
- PAtest: mwqncWSnzUzouPZcLQWcLTPuSVq3rSiAAa - 92oB4Eb4GBfutvtEqDZq3T5avC7pnEkPVme23qTb5mDdDesinm6

#### Card transfer tag generation

Generated from deck spawn txnid ....

## Drawbacks

Why should we *not* do this?

## Alternatives

What other designs have been considered? What is the impact of not doing this?

## Unresolved questions

What parts of the design are still to be done?
