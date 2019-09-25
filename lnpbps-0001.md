```
LNPBPS: 0001
Layer: Transactions (1)
Field: Cryptographic primitives
Title: Cryptographic commitments with public key tweaking
Author: Dr Maxim Orlovsky <orlovsky@pandoracore.com>
Based on: <https://blog.eternitywall.com/2018/04/13/sign-to-contract/>, 
          <https://github.com/sipa/bips/blob/bip-schnorr/bip-taproot.mediawiki#tagged-hashes>
Comments-URI: 
Status: Draft
Type: Standards Track
Created: 2019-23-09
License: CC0-1.0
```

## Abstract

## Background

Pay-to-contract cryptographic commitments has been previously described in many publications, such as 
[Eternity Wall's "sign-to-contract" article](https://blog.eternitywall.com/2018/04/13/sign-to-contract/).


## Motivation

While pay-to-conctract key tweaking is already widely used practice 
([OpenTimeStamps](https://petertodd.org/2016/opentimestamps-announcement), 
[merchant payments](https://arxiv.org/abs/1212.3257)) and will be even more adopted with the introduction of 
[Taproot](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2018-January/015614.html), 
[Graphroot](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2018-February/015700.html), 
[MAST](https://github.com/bitcoin/bips/blob/master/bip-0114.mediawiki) and many other proposals, the hash, used for 
public key tweaking under one standard (like this one) can be provided to some uninformed third-party as a commitment 
under the other standard (like Taproot), and there is non-zero chance of a collision, when serialized data will present 
at least a partially-valid Bitcoin script or other code that can be interpreted as a custom non-OpenSeals data 
and used to attack that third-party. In order to reduce the risk, we follow the practice introduced in the 
[Taproot BIP proposal](https://github.com/sipa/bips/blob/bip-schnorr/bip-taproot.mediawiki#tagged-hashes) of prefixing 
the proof hash with *two* hashes of OpenSeals-specific tags, namely "openseal". Two hashes are required to 
further reduce risk of undesirable collisions, since nowhere in the Bitcoin protocol the input of SHA256 starts with 
two (non-double) SHA256 hashes <https://github.com/sipa/bips/blob/bip-schnorr/bip-taproot.mediawiki#tagged-hashes>.


## Specification

Cryptographic commitment made with pay-to-contract scheme for a given output `n` in a transaction `T` SHOULD BE 
considered valid if, and only if:

1. The `n`th output pays an arbitrary amount of satoshis to `P2PKH`, `P2WPKH` or `P2SH`-wrapped `P2WPKH`.
2. The public key of this output is tweaked using the method described below

Otherwise, the if MUST BE considered as an invalid and MUST NOT BE accepted. 

The whole algorithm thus looks in the following way:
1. Compute SHA256 hash of some consensus-serialized data which we would like to commit to (**commitment**): 
   `commitment = SHA256(serialized_data)`
1. Get result of `hash(message, tag) := SHA256(SHA256(tag) || SHA256(tag) || message)` function
   (see [Taproot BIP](https://github.com/sipa/bips/blob/bip-schnorr/bip-taproot.mediawiki#tagged-hashes) for the details),
   where `message` MUST contain concatenated original public key `original_pubkey` and `commitment`:
   `h = hash(original_pubkey || id, tag)`
2. Compute `new_pubkey = original_pubkey + h * G`
3. Compute the address as a standard Bitcoin `P2(W)PKH` using `new_pubkey` as public key

In order to be able to spend the output later, the same procedure should be applied to the private key.

## Compatibility

## Rationale

### Use of addition instead of multiplication

TBD

### Selection of supported transaction output types

Rationale for not supporting other types of transaction outputs for the proof commitments:
* `P2PK`: considered legacy and MUST NOT be used due to a possible vulnerability to a future quantum computing attacks;
* `P2WSH`: there is no simple, universal and robust way to deterministically define which of the public keys present 
   inside the script and which of them are used for the commitment, however support to P2(W)SH may be added with a
   future proposal;
* `P2SH`, except `P2SH`-wrapped `P2WPKH`, but not `P2SH`-wrapped `P2WSH`: the same reason as for `P2WSH`;
* `OP_RETURN` outputs can't be tweaked, since they do not contain a public key and serve pre-defined purposes only. 
   If it is necessary to commit to OP_RETURN output one should instead use [LNPBPs-0001](/llnpbps-0002.md)
*  Non-standard outputs: tweak procedure can't be standardized.

## Reference implementations

## Acknowledgements

## References

## Copyright

## Test vectors
