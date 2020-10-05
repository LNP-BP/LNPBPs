```
LNPBP: 0001
Layer: Transactions (1)
Vertical: Bitcoin protocol
Title: Key tweaking: collision-resistant elliptic curve-based commitments
Authors: Dr Maxim Orlovsky <orlovsky@protonmail.ch>,
         Rene Pickhardt,
         Federico Tenga,
         Martino Salvetti,
         Giacomo Zucco,
         Max Hillebrand,
         Christophe Diederichs
Comments-URI: https://github.com/LNP-BP/lnpbps/issues/3
Status: Proposal
Type: Standards Track
Created: 2019-09-23
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

1. Construct a byte string `lnbp1_msg`, composed of the original message prefixed with a single SHA256 hash of `LNPBP1`
   string and a single SHA256 hash of protocol-specific tag:
   `lnbp1_msg = SHA256("LNPBP1") || SHA256(<protocol-specific-tag>) || msg`
2. Compute HMAC-SHA256 of the `lnbp1_msg` and `P`, named **tweaking factor**: `f = HMAC_SHA256(lnbp1_msg, P)`
3. Make sure that the tweaking factor is less than order `p` of Zp prime number set used in Secp256k1 curve; otherwise
   fail the protocol.
3. Multiply the tweaking factor on Secp256k1 generator point `G`: `F = G * f` ignoring the possible overflow of the
   resulting elliptic curve point `F` over the order `n` of `G`. Check that the result not equal to the
   point-at-infinity; otherwise fail the protocol, indicating the reason of failure, such that the protocol may be run
   with another initial public key `P'` value.
4. Add two elliptic curve points, the original public key `P` and tweaking-factor based point `F`, obtaining the
   resulting tweaked public key `T`: `T = P + F`. Check that the result not equal to the point-at-infinity; otherwise
   fail the protocol, indicating the reason of failure, such that the protocol may be run with another initial
   public key `P'` value.

The final formula for the commitment is:
`T = P + G * HMAC_SHA256(SHA256("LNPBP1") || SHA256(<protocol-specific-tag>) || msg, P)`

**Reveal procedure** over the commitment (the tweaked public key `T`) can be performed with the provision of the
original public key `P` and message `msg` (assuming that the verifying party is aware of the protocol-specific tag
and `LNPBP1` tag) and runs as follows:

1. Make sure that the provided tweaked public key `T` lies on the elliptic curve and is not the point at infinity.
2. Compute `T' = P + G * HMAC_SHA256(SHA256("LNPBP1") || SHA256(<protocol-specific-tag>) || msg, P)` according to the
   rules of the **commit procedure** described above.
3. Make sure that `T' = T`, otherwise fail the procedure

Alternatively, verifier may be provided with the tweaking factor (value of HMAC of the tagged message) `f` and the
message `msg`. In this case the procedure will be:

1. Compute `P = T - f * G`, i.e. invert the public key `F = f * G` on the x-axis and obtain value for `-F`; then
   add `-F` and `T` which will give the value of `P`: `-F + T = P`
2. Compute `T' = P + G * HMAC_SHA256(SHA256("LNPBP1") || SHA256(<protocol-specific-tag>) || msg, P)` according to the
   rules of the **commit procedure** described above.
3. Make sure that `T' = T`, otherwise fail the procedure

The second option of a commitment proof consisting of the tweaking factor `f` (256-bit HMAC value) has a bigger
computational overhead, however the proof may be serialized utilizing one byte less, since the public key `P`
requires 33 bytes for serialization (in the compressed form). Alternatively, if the key `P` is selected and serialized
according to the Secp256k1 Schnorr's signature rules [4] the first reveal scheme will utilize the same proof storage
space and will be more advantageous.


## Compatibility

The proposed procedure should be compatible with previously-created pay-to-contract-style commitments based on SHA256
hashes under an assumption of SHA256 collision resistance. Utilization of duplicated protocol tag hash prefix guarantees
randomness in the first 64 bytes of the resulting tweaking string `s`, reducing probability for these bytes to be
interpreted as a correct message under one of the previous standards.

The proposal is compatible with Schnorr and Taproot-related standards because it relies on the same tagged hash prefix
approach [4, 5].


## Rationale

### Use of HMAC: prevention of length extension attacks and multiple commitments

### Use of protocol tags: prevention of inter-protocol collision attacks

The use of duplicated protocol-specific tag hash is originally proposed by Peter Wuille in [4] and [5] in order to
prevent potential reply-attacks for interpreting message under different protocols. The choice of duplicate single
SHA256 hash is made due to the fact that according to Peter Wuille it is not yet used in any existing bitcoin protocol,
which increases compatibility and reduces chances of collisions with existing protocols.

### Protocol failures

The protocol may fail during three of **commitment* procedure steps:
* when the *tweaking factor* `f` exceeds Zp order `p` for Secp256k1
* when the multiplication of Secp256k1 generator point `G` on the *tweaking factor* `f` results in `F` equal to the
  point at infinity
* when additions of the point `F` and the original public key `P` results in `T` equal to the point at infinity

The probabilities of these failures are infinitesimal; for instance the probability of SHA256 hash value of a random
message exceeding Zp order `p` is `(2^256 - p) / p`, which is less than `3.7*10^-66`, i.e. lower than probability of CPU
failure. The probability of the second and third failure is even lower, since the point at infinity may be obtained only
if `F` will be equal to `-G` or `-P`, i.e. the probability of private key collision, equal to the inverse of Secp256k1
curve generator point order `n`.

These cases may be ignored by a protocol user -- or, alternatively, in case of the protocol failure the user may
change `P` value and re-run the protocol.

### No nonce

Nonce is not required since the original public key, hidden in the resulting tweaked key, presents sufficient entropy
for an attacker to be not able to guess the original message even for short and standard messages.


## Reference implementation

<https://github.com/LNP-BP/rust-lnpbp/blob/master/src/cmt/pubkey.rs>


## Acknowledgements

Authors would like to thank:
* Alekos Filini for his initial work on the commitment schemes as a part of early RGB effort [6];
* ZmnSCPxj for bringing attention to possible Taproot-compatibility issues [7];
* Peter Wuille for a proposal on the tagged hashes, preventing reply-type of attacks [5];
* Authors of Sidechains whitepaper for paying attention to the potential length-extension attacks and the introduction
  of HMAC-based commitments to the original public key [3];


## References

1. Ilja Gerhardt, Timo Hanke. Homomorphic Payment Addresses and the Pay-to-Contract Protocol. arXiv:1212.3257 \[cs.CR\]
   <https://arxiv.org/pdf/1212.3257.pdf>
2. [Eternity Wall's "sign-to-contract" article](https://blog.eternitywall.com/2018/04/13/sign-to-contract/)
3. Adam Back, Matt Corallo, Luke Dashjr, et al. Enabling Blockchain Innovations with Pegged Sidechains (commit5620e43).
   Appenxix A. <https://blockstream.com/sidechains.pdf>.
4. Pieter Wuille. Schnorr Signatures for secp256k1.
   <https://github.com/sipa/bips/blob/bip-schnorr/bip-schnorr.mediawiki#Public_Key_Generation>
5. Pieter Wuille. Taproot: SegWit version 1 output spending rules.
   <https://github.com/sipa/bips/blob/bip-schnorr/bip-taproot.mediawiki#Specification>
6. RGB Protocol Specification, version 0.4. "Commitment Scheme" section.
   <https://github.com/rgb-org/spec/blob/old-master/01-rgb.md#commitment-scheme>
7. <https://github.com/rgb-org/spec/issues/61>


## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.


## Test vectors

TBD
