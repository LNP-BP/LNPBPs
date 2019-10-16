```
LNPBPS: 0001
Layer: Transactions (1)
Field: Cryptographic primitives
Title: Key tweaking: collision-resistant elliptic curve-based commitments
Author: Dr Maxim Orlovsky <orlovsky@pandoracore.com>
Comments-URI: https://github.com/LNP-BP/lnpbps/issues/1
Status: Draft
Type: Standards Track
Created: 2019-23-09
License: CC0-1.0
```

## Abstract

The work proposes a standard for cryptographic commitments based on elliptic curve properties, that can be embedded
into Bitcoin transaction without additional storage footprint. This commitments are private: the can be detected and 
revealed only to the parties sharing some secret (original value of the public key).

The proposed standard combines all existing best practices and solves problems of previous approaches, like 
length-extension attacks, cross-protocol reply attacks or public key value overflows.


## Background

Cryptographic commitments represent a way to commit to some message without revealing it. The procedure
consists of two phases, **commit** and **reveal**. In the *commit* phase a party (**committer**) willing to prove its 
knowledge of some message computes a *cryptographic hash function* over that message, producing a message **digest**, 
which can be provided to other party(ies). In the *reveal* phase *committer* reveals the actual message, and each party 
accessing it may check that it's hash is equal to the originally provided *digest*.

Key tweaking is a procedure of cryptographic commitment to some **message** using elliptic curve properties. The 
procedure takes public and private key pair (called **original keys**) and returns a **tweaked key** pair containing
the commitment. The tweaked key pair can't be distinguished from any other elliptic keys; this property allows
to hide the actual commitment in such a way that it can be only known to those parties which have knowledge of a
secrets: original key pair.

This type of commitment was originally proposed as a part of "pay to contract" concept by Ilja Gerhardt and Timo Hanke
in [1] and later used by Eternity Wall [2] for the same purpose. However, these proposals were vulnerable for 
length-extension attacks; it was also opened to the problem of the message digest exceeding the elliptic curve order,
which may lead to protocol failures. These problems were fixed as a part of sidechain-design effort by Blockstream [3], 
which proposed to utilize HMAC function and introduced concept of nonce, that can be changed in order to find a digest
value which will not exceed the elliptic curve order.

Here we propose an algorithm based on Blockstream work, enhanced with Pieter Wuille Tagged Hashes procedure, coming from
a specification on Schnorr signatures in Bitcoin [4], also used in Taproot proposal [5]. This procedure prevents
cross-protocol collisions, such that original message byte sequence can't be reinterpreted under other protocol.


## Motivation

Publication of cryptographic commitments to Bitcoin blockchain is a widely used mechanism, allowing timestamping of the
commitment: it can be used to prove the fact that some information was known before certain period in time without
revealing the actual information. Use of elliptic curve homomorphic properties allows to perform such commitments
without increasing the size of the transaction, by leveraging existing transaction outputs and not polluting blockchain
space with excessive OP_RETURN's. However, as of today, there is no single standard for such commitments, while
different practices, used for the purpose (see [1, 2, 3]) contain multiple collision risks, such as possibility of the 
length-extension attacks, cross-protocol reply attacks or protocol failures due to elliptic curve order "overflow"
(more details are given in the [Background](#Background) section). This standard combines existing best practices into 
a single algorithm, that avoids all of those issues.


## Specification

For a given message `msg` and original public key `P` the **commit procedure** is defined as follows:

1. Compute HMAC-SHA256 of the `msg` and `P`: `hmac = HMAC_SHA256(msg, P)`
2. Define a protocol-specific unique tag `tag` and compute concatenation of its two single SHA256 hashes 
   (after a duplicated tag procedure from Peter Wuille's tagged hash function [4, 5]): `t = SHA256(tag) || SHA256(tag)`
3. Compute a random 8-bit nonce `n`
4. Compute tweaking string `s = t || SHA256('LNPBPS-0001') || hmac || n`. The length of the string MUST BE 129 bytes.
5. Compute tweaking factor `h = SHA256(s)`
6. If the factor value is equal of greater than the elliptic curve order repeat from step 3 with a different nonce
7. Compute tweaked public `T = P + h * G` and (if necessary) private key `t = p + h` according to standard 
   elliptic-curve private and public key addition procedures.

**Reveal procedure** must be provided with the value of the original public key `P`, nonce `n`, message `msg` and runs 
as follows:

1. Make sure that the provided tweaked public key `T` lies on the elliptic curve
2. Compute HMAC-SHA256 of the `msg` and `P`: `hmac = HMAC_SHA256(msg, P)`
3. Compute duplicated SHA256 of the protocol tag `t = SHA256(tag) || SHA256(tag)`
4. Compute tweaking string `s = t || SHA256('LNPBPs-0001') || hmac || n`. The length of the string MUST BE 129 bytes.
5. Compute tweaking factor `h = SHA256(s)`
6. If the factor value is equal of greater than the elliptic curve order fail the reveal procedure
7. Compute tweaked public key `T' = P + h * G`. Fail the procedure if it does not match the provided tweaked key `T`.


## Compatibility

The proposed procedure should be compatible with previously-created pay-to-contract-style commitments based on SHA256 
hashes under an assumption of SHA256 collision resistance. Utilization of duplicated protocol tag hash prefix guarantees
randomness in the first 64 bytes of the resulting tweaking string `s`, reducing probability for these bytes to be
interpreted as a correct message under one of the previous standards.

The proposal is compatible with Schnorr and Taproot-related standards because it relies on the same tagged hash prefix 
approach [4, 5].


## Rationale

### Use of EC point addition instead of multiplication

### Use of HMAC: prevention of length extension attacks and multiple commitments

### Use of protocol tags: prevention of inter-protocol collision attacks

The use of duplicated protocol-specific tag hash is originally proposed by Peter Wuille in [4] and [5] in order to 
prevent potential reply-attacks for interpreting message under different protocols. The choice of duplicate single 
SHA256 hash is made due to the fact that according to Peter Wuille it is not yet used in any existing bitcoin protocol, 
which increases compatibility and reduces chances of collisions with existing protocols.


### Use of nonce


## Reference implementations

<https://github.com/LNP-BP/rust-lnpbp/blob/master/src/commitments/secp256k1.rs>

## Acknowledgements

Authors would like to thank:
* Alekos Filini for his initial work on the commitment schemes as a part of early RGB effort [6]; 
* ZmnSCPxj for bringing attention to possible Taproot-compatilibity issues [7];


## References

1. Ilja Gerhardt, Timo Hanke. Homomorphic Payment Addresses and the Pay-to-Contract Protocol. arXiv:1212.3257 [cs.CR] 
   <https://arxiv.org/pdf/1212.3257.pdf>
2. [Eternity Wall's "sign-to-contract" article](https://blog.eternitywall.com/2018/04/13/sign-to-contract/)
3. Adam Back, Matt Corallo, Luke Dashjr, et al. Enabling Blockchain Innovations with Pegged Sidechains (commit5620e43).
   Appenxix A. <https://blockstream.com/sidechains.pdf>.
4. Pieter Wuille. Schnorr Signatures for secp256k1.
   <https://github.com/sipa/bips/blob/bip-schnorr/bip-schnorr.mediawiki#Public_Key_Generation>
5. Pieter Wuille. Taproot: SegWit version 1 output spending rules.
   <https://github.com/sipa/bips/blob/bip-schnorr/bip-taproot.mediawiki#Specification>
6. <https://github.com/rgb-org/spec/blob/old-master/01-rgb.md#commitment-scheme>
7. <https://github.com/rgb-org/spec/issues/61>


## Copyright

## Test vectors
