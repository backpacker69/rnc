# Transaction test vectors

- Status: work in progress
- Type: test vectors
- Related components: transaction-specification
- Start Date: 28-10-2016

## Summary

The PeerAssets specification is just a protocol, multiple implementations exist.
Therefore, it is crucial that a set of test vectors is defined that can be used to validate an implementation.
Only implementations that pass all of the tests specified in the latest version of this article can be considered valid.

## Test vectors

These test vectors are generated using the [PeerScript-labs][1] prototype implementation.
Small inline code snippets are included to clarify the test procedure.

### P2TH tags

The P2TH addresses for the transfer transactions are based on the transaction id of the deck spawn transaction (= asset id).
The test vector is generated using bitcoin-tool and tested using bitcore-ppc.
As both tools agree, the test vector is considered valid.
Note that the compressed public key version of the address is used!

Test vector:
```
{
  "txnId": "c23375caa1ba3b0eec3a49fff5e008dede0c2761bb31fddd830da32671c17f84",

  "WIF": "UBctiEkfxpU2HkyTbRKjiGHT5socJJwCny6ePfUtzo8Jad9wVzeA",
  "Address":"PRoUKDUhA1vgBseJCaGMd9AYXdQcyEjxu9"
  "testnetWIF": "cU6CjGw3mRmirjiUZfRkJ1aj2D493k7uuhywj6tCVbLAMABy4MwU",
  "testnetAddress": "mxjFTJApv7sjz9T9a4vCnAQbmsqSoL8VWo"
}
```

bitcore-ppc test code:
```
var binaryTxnId = Buffer.from(testVector.txnId, 'hex');
var bn = crypto.BN.fromBuffer(binaryTxnId);
var priv = new PrivateKey(bn);
assert.equal(priv.toWIF(), testVector.WIF);
assert.equal(priv.toAddress().toString(), testVector.Address);
```

bitcoin-tool test code:
```
bitcoin-tool
  --input c23375caa1ba3b0eec3a49fff5e008dede0c2761bb31fddd830da32671c17f84
  --input-type private-key
  --input-format hex
  --network peercoin
  --output-type all
  --public-key-compression compressed
```

<!-- References -->
[1]: https://hrobeers.github.io/peerscript-labs/
