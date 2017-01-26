# PeerAssets Alias/Proof-of-identity protocol specification

- Status: proposed
- Type: new feature
- Related components: 0003-peerassets-alias-poid-transaction-specification.proto
- Start Date: 13-01-2017

## Summary

Extension of PeerAssets protocol specifies using singlet PeerAsset deck to alias the Peercoin address with any UTF-8 string with added secondary functionality of using descriptive information contained in the PeerAssets token to allow proof-of-identity verification of the privkey owner.

Use of aliases in the form of @alias instead of a standard Peercoin address is made possible for all PeerAssets and Peercoin transactions. A blockchain client (wallet) can save verified alias locally and translate @Bob to a Peercoin address like PDF4NNGBGRSxnRJPyNQyMHqffqbXEtRy5T on the fly.

This proposal tries to come up with an elegant solution to the problem of matching the "real life" identity of a person with an address on the blockchain by verifying that specific public key is in control of person X. This protocol allows knowing your customer (KYC) and obliging to the law in some countries. 1 Usage of this protocol for identity verification also prevents sybil attack if there is a service depending on trusting some specific key set.

Unlike similar protocols for blockchain proof-of-identity, this protocol allows an alias to be transferable to another address, providing both improved privacy and utility. Mobility of the alias is possible as the alias is a simple PeerAssets "card," which means it is as mobile as any other blockchain token issued via the PeerAssets protocol. Aliases can be transferred, traded, updated or even destroyed on the blockchain.

Verification of the alias works just like verifying email upon registration to some online forum, where the claim of identity is backed by proving ownership of the email address. A similar process is employed here, where the party claiming the identity behind the Peercoin address backs it by posting proof on their social media account or in a private exchange via email.
Two modes of verification are possible: manual and automatic.

Manual verification would be similar to the standard message signing process where a person proving the identity or ownership of the address is signing the text message with their private key and providing the interested party with the computed signature, which can be checked against their public key. [2]
Automatic verification would be executed by the software client at the moment of the alias being assigned to an address. Upon creation of the PeerAsset alias token, the client would post a message and the signature publicly to Twitter, Reddit, Facebook or any other social media website supported by the protocol via their public API. Posting the message and signature publicly allows anyone to verify that a specific social media website user is indeed the owner of that specific PeerAsset token. Automation can work in the other direction as well, for example: A protocol aware client would query the Facebook API for *$facebook_id* and check if the Facebook name matches the *full_name* attribute of a PeerAsset token.

The protocol supports validation of multiple social media accounts, so the user can get authenticated on multiple platforms. The more platforms are verified, the greater the chance that the alias owner is claiming his true identity.
Having such a setup also allows aliasing the username of a social media account to a Peercoin account, allowing seamless monetary transactions via Facebook for example.

_______

## Detailed design

### Alias

An alias is a substitute string which is translated to a Peercoin address according to publicly recorded information. Each protocol aware client should be able to understand this and translate an @alias string to a valid Peercoin address on the fly.

Alias information is encoded in the *name* field of *deck_spawn* PeerAssets transaction.

### Deck spawn

An alias deck is spawned like any other PeerAssets deck. However, it is a single asset deck (singlet deck), so it is spawned with *issue_mode* = `SINGLET`. The name of a deck is the `alias` and is allowed to be any UTF-8 string desired by the issuer.

```
deck_spawn_metainfo =
		 {
		"version" : 1,
		"name": "bob",
		"number_of_decimals": 0,
		"issue_mode": SINGLET
		}
```

Unlike the standard deck_spawn PeerAssets transaction, the alias deck_spawns pay to a second P2TH address which is used to distinguish the identity decks from regular PeerAssets decks. Secondary P2TH tag is used to signal the software client to parse the alias metainfo attached to the token. Naturally, the two types of decks are still compatible.

```
vout[0] = deck_spawn P2TH
vout[2] = alias P2TH
```

#### Alias deck P2TH

> Address: `PP1D3UAXSiKztcMFWwscuTL8rdFhbRkw3z`

> WIF: `U789GmLYDNeJqVQPMmbJn2Jba7NbjhRRMjtfrBWHPpVgZU1yGCY6`

### Card issue

From the spawned deck, the issuer sends a single card to the new owner. This process is referred to as "*alias assigning*". Alias assigning assigns the alias and other metainfo to the receiver Peercoin address, making the receiver address synonymous with the card metainfo. The recipient address becomes the holder of alias "bob."

Identity assigning is extended PeerAssets *card_transfer* transaction, where identity metainfo is attached to *asset_specific_data* field in the card_transfer.

Version 1 of the identity protocol proposes the following fields: version, full name, Email address, Twitter username, Github username, Reddit username, Freenode nick, Facebook id, GPG key fingerprint and var.
Bob is allowed to add fields until 80bytes maximum size is reached.

* `version`
   uint32; signals the client to parse the message according to standard of version n.

* `full_name`
  UTF-8 string; Full name of this person.

* `email`
  UTF-8 string; Email address in format of user@provider.domain.

* `twitter_id`
  UTF-8 string; Twitter.com username, in "@username" format.

* `github_id`
  UTF-8 string; Github.com username.

* `freenode_id`
  UTF-8 string; Freenode.net IRC nick.

* `facebook_id`
  uint32; 10 digit Facebook.com username ID number.

* `gpg_fingerprint`
  bytes; GPG pubkey fingerprint in bytes format.

* `var` (optional)
  bytes; Any additional data. 

Example of identity metainfo:

```
identity  = {
	"full_name": "Bob Pando",
	"email: "bob@protonmail.com",
	"twitter_id": "@bob911",
	"github_id": "bob911",
	"reddit_id": "bobbyg16",
	"facebook_id": 1086086247, 
	"gpg_fingerprint": b"",
	"freenode_id": "",
	"var": ""
	}
```

#### Substitutes

User does not have to prefix the Twitter username with the `@` , client is expected to do so automatically.
As the name of this deck is "bob", $n can be used as a subsitute for *deck_name* and automatically replaced by *deck_name* in each string.

_Substitutes:_

* `$e` - Email

* `$n` - Deck name

* `$fn` - Full name

* `@` is automatically prefixed for Twitter username.

Valid identity_metainfo with these rules applied:

```
identity = {
	"full_name": "Bob Pando",
	"email": "$n@protonmail.com",
	"twitter_id": "$n911",
	"github_id": "$n911",
	"reddit_id": "$nbyg16",
	"facebook_id": 1086086247, 
	"gpg_fingerprint": b"",
	"freenode_id": "",
	"var": ""
	}
```

The size of the identity metainfo without applying the substitutes is 64 bytes, while applying them saves 6 bytes and counts 58 bytes in total. Substitutes allow Bob to pack more information in the message.

### Alias transfer

Transferring the alias to a different address is a process equal to the standard PeerAssets *card_transfer* transaction. The most recent *card_transfer* of the identity deck is considered an upgrade over all previous ones, thus being the only relevant one.
To transfer full control over the alias deck, an issuer should use the PeerAssets deck_transfer transaction to transfer the deck to another party.

### Revoking alias

Revoking alias is equal to PeerAssets *burn_transaction* and is done by sending the token back to *deck_spawn* address.

### Proof-of-identity

Proof of identity verification process trusts the social media sites to force the user to use their true identity. Facebook is notorious for doing this, as they force their users to deliver copies of government issued identity documents. Twitter also allows user verification via identity documents like passports.
As mentioned in the summary, the protocol should allow two modes of operation when it comes to proof-of-identity: manual and automatic. Both are based on the same principle of public-key cryptography digital signature in which a message is signed with the sender's private key and can be verified by anyone who has access to the sender's public key. This verification proves that the sender had access to the private key, and therefore is likely to be the person associated with the public key. [3]
The party proving the ownership of the address (public key) signs the message and posts the signature publicly on their social media account, so it can be verified against the publicly known pubkey associated with the PeerAssets alias token.
This process can be done manually per request from the interested party, or automated via dedicated software which will handle both the *deck_spawn*, *card_issue* and logging to social media sites to post the signature. Such software should be able to automatically use the public API of social media sites to locate the post containing the signature and verify it against the public key upon making contact with some alias token.

### Revoking proof-of-identity

Revoking proof-of-identity is done by burning the token by sending it to the *deck_spawn* address, which is a process known as PeerAssets card *burn_transaction*.

### Alias squatting

As multiple PeerAssets decks can have the exact same name with the exact same @alias handle, alias squatting and identity theft is possible due to this. However, no PeerAssets deck will have the same asset_id, so this problem can be circumvented by the original holder publicly stating which deck is actually issued by him. Beside, no squatter will be able to use the Twitter, Email or Facebook with the same name as the alias to verify the alias, except their actual owner.
Future users are advised to do the alias verification manually and save the autohorized decks in the local wallet (local alias address book).

## Advantages

* Improving user experience by providing a way to use human-friendly names (aliases) instead of Peercoin addresses

* Improved quantum resistance by allowing users to discard key set often and alias the new one without the hastle of having to tell every business partner / transacting peer that address has been changed.

## Drawbacks

* Alias squatting is possible

## Alternatives

## Unresolved questions

<!-- References -->

[1]: https://en.wikipedia.org/wiki/Know_your_customer
[2]: https://bitcointalk.org/index.php?topic=990345.0
