```
LNPBPS: 0003
Layer: Transaction (1)
Field: Cryptographic primitives
Title: Deterministic definition of transaction output containing cryptographic commitment
Author: Dr Maxim Orlovsky <orlovsky@pandoracore.com>
Comments-URI: https://github.com/LNP-BP/lnpbps/issues/5
Status: Draft
Type: Standards Track
Created: 2019-27-10
License: CC0-1.0
```

## Abstract

The standard defines an algorithm for deterministic definition of the transaction output containing cryptographic
commitments made under the standards LNPBPS-1 [1] and LNPBPS-2 [2].


## Background

Embedding cryptographic commitments into Bitcoin blockchain has become a common practice [3]. Bitcoin blockchain provides
strong guarantees on the underlying data immutability, presenting a reliable, distributed and censorship-resitent 
proof-of-publication [4] medium. While existing standards defines how the cryptographical commitments can be used with
public keys on Secp356k1 elliptic curve (used by Bitcoin) and embedded withing transaction outputs, it still has to be
defined how the interested parties may detect which of a Bitcoin transaction output contains the commitment, presenting
an interest to the party under some given protocol.


## Motivation

A number of use cases for cryptographic commitments (CC) require some commitment to be unique. For instance, single-use
seals [5] may utilize them to achieve it's one-time commitment properties, as may be required in some of the 
applications [6], [7]. While the deterministic way for embedding into Bitcoin transactions (like "always use the first 
output) may be defined on a per-protocol basis, it seems that some best-practices, defining the best privacy-preserving
and secure algorithm for such deterministic commitments in Bitcoin transactions may also be useful for the industry.

Some of the protocols relies on the first OP_RETURN output present in the transaction to contain the commitment [3].
However, this would not work for commitments supporting other output types ([2], [8]), and for may introduce collisions
with other protocols using OP_RETURN, like OMNI Layer [9].

Static definition of the output containing the commitment will be incompatible with BIP-... [10], defining deterministic
transaction output ordering and will not work with Lightning Network, following the same rules [11].


## Specification

Cryptographic commitments under this standard are made according to LNPBPS-1 [1] and LNPBPS-2 [2] standards.

To deterministically define the number of transaction output that contains CC under some protocol `P`, the commiting
party "Alice" `A` (which may be represented by a single person or some m-of-n federation) and the verifying  party "Bob" 
`B` (which may be represented by a single person or some m-of-n federation) need to agree  on three parameters:
1. A protocol-defined parameter `s`, acting as a seed for all protocol-wide commitments, this must be a 8-bit value;
2. A commitment-specific parameter `c`, which should not be directly reflected in the transaction, containing a giver CC.
   This parameter under some circumstances, beforehand agreed by the parties, may be absent. The value of `c` must be a
   8-bit value.

These two parameters serve as a "salt" for the deterministic procedure, preventing third-parties from guessing the 
information of the actual commitment, thus avoiding possible censorship by the miners and on-chain based transaction
analysis.

Alice, creating the transaction containing CC, and Bob, verifying it, must use the following procedure:
1. Compute the transaction fee `f`, as a difference between total transaction output amounts in satoshis and total
   inputs amounts, in satoshis: `f = sum(outputs) - sum(inputs)`. According to the Bitcoin consensus rules, this amount
   must always be positive and greater than zero(`f > 0`); otherwise it is not a valid Bitcoin transaction and the 
   protocol must fail.
2. Compute the adjusted fee `a` as a 32-bit modulo of the potentially-64 bit number `f`: `a = f mod 2^32`.
3. Add to this number a previously-agreed values of `s` and `c`, or, if `c` was not defined, use `0` for `c` value
   by default. This will give a commitment-factor `x = a + s + c`. Since `s` and `c` is a 8-bit numbers and `a` is a
   32-bit number, the result MUST BE a 64-bit number, which will prevent any possible number overflows.
4. Get the number of outputs `n` for the transaction containing the output with the given cryptographic commitment
5. Compute `d` as `d = n mod x`. The `d` will represent a number of transaction output which MUST contain a cryptographic
   commitment. All other transaction outputs under this protocol MUST NOT contain cryptographic commitments.


## Compatibility

The proposed standard is incompatible with the existing practices for definition of the transaction output containing 
cryptographic commitment, namely:
1. OpenTimestamps, putting the commitment into the first transaction output [1]
2. ...

Nevertheless, use of protocol-specific pre-defined salt may be utilised as a flag signalling the support of the current
standard, which will help to avoid possible commitment collisions across different protocols.


## Rationale

The rationale for the technical decision is provided within the specification text.


## Reference implementation

<https://github.com/LNP-BP/rust-lnpbp/blob/master/src/commitments/tx.rs>


## Acknowledgements

Authors would like to thank  Giacomo Zucco and Alekos Filini for their initial work on the commitment schemes as a part 
of early RGB effort [7].


## References

1. Maxim Orlovsky. Key tweaking: collision-resistant elliptic curve-based commitments (LNPBPS-1 Standard). 
   <https://github.com/LNP-BP/lnpbps/blob/master/lnpbps-0001.md>
2. Maxim Orlovsky. Deterministic embedding of elliptic curve-based commitments into transaction outputs (LNPBPS-2 
   standard). <https://github.com/LNP-BP/lnpbps/blob/master/lnpbps-0002.md>
3. Peter Todd. OpenTimestamps: Scalable, Trust-Minimized, Distributed Timestamping with Bitcoin. 
   <https://petertodd.org/2016/opentimestamps-announcement>
4. Peter Todd. Setting the record straight on Proof-of-Publication.
   <https://petertodd.org/2014/setting-the-record-proof-of-publication>
5. Peter Todd. Preventing Consensus Fraud with Commitments and Single-Use-Seals.
   <https://petertodd.org/2016/commitments-and-single-use-seals>
6. Peter Todd. Scalable Semi-Trustless Asset Transfer via Single-Use-Seals and Proof-of-Publication.
   <https://petertodd.org/2017/scalable-single-use-seal-asset-transfer>
7. OpenSeals Framework <https://github.com/rgb-org/spec/blob/v1.0/01-OpenSeals.md>
8. OMNI
9. RGB
10. BIP deterministic output ordering
11. LN transactions

## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.


## Test vectors

TBD