```
LNPBPS: 0002
Layer: Transactions (1)
Field: Cryptographic commitments
Title: Deterministic embedding of elliptic curve-based commitments into transaction outputs
Author: Dr Maxim Orlovsky <orlovsky@pandoracore.com>
Comments-URI: https://github.com/LNP-BP/lnpbps/issues/4
Status: Draft
Type: Standards Track
Created: 2019-27-10
License: CC0-1.0
```

## Abstract

The standard defines an algorithm for deterministic embedding and verification of cryptographic commitments based
on elleptic-curve public and private key modifications (tweaks) inside the existing types of Bitcoin transaction output
and Bitcoin scripts.


## Background

Cryptographic commitments embedded into bitcoin transactions is a widely-used practice. It's application include
timestamping [1], single-use seals [2], pay-to-contract settlement schemes [3], sidechains [4], blockchain anchoring [5],
Taproot, Graftroot [6, 7, 8], Scriptless scripts [9] and many others. 


## Motivation

A number of cryptographic commitment (CC) use cases require the commitment to be present within non-P2(W)PK(H) outputs. 
For instance, embedding CC into Lightning Network (LN) payment channel state updates in the current version require 
modification of an offered HTLC and received HTLC transaction outputs [10], and these transactions contain only a single 
P2WSH output. With the following updates [11] LN will likely will change a single P2WPKH (named `to_remote`) output 
within the commitment transaction, also leading to a requirement for a standard and secure way of making CC inside 
P2(W)SH outputs.

At the same time, P2(W)SH and other non-standard outputs (like explicit P2S outputs, including OP_RETURN type) are not
trivial to use in CC commitments, since CC requires deterministic definition of the actual commitment case. Normally,
one of the most secure and standard CC scheme uses homomorphic properties of the public keys [12]; however multiple
public keys may be used within non-P2(W)PK(H) output. Generally, these outputs present a hash of the actual script,
hiding the information of the used public key or their hashes. Moreover, it raises the question of how the public
key within Bitcoin Script may be defined/detected, since it is possible to represent the public key in a number of 
different ways within bitcoin script itself (explicitly or by a hash, and it's not trivial to understand where some
hash stands for a public key or other type of preimage data). This all raises a requirement to define a standard way and
some best practices for CC in non-P2(W)PK(H) outputs. This proposal tries to address the issue by proposing a common 
standard on the use of public-key based CC [12] withing all possible transaction output types.


## Specification

In order to create a deterministic way for CC within transaction output first we have to address two main issues:
1. Create a definition for what is a public key that may be used for a CC for all possible types of transaction outputs.
2. Define which of the public keys has to participate in a given CC for the outputs that may contain more than a single
   public key.
   
### Deterministic definition for a cryptographically-committable public key fingerprint within transaction output

A public key fingerprint within a given bitcoin script bytecode is defined as anything satisfying any of the following
conditions:
1. A topmost stack element before an execution of Bitcoin script matching the pattern 
   `OP_HASH160 OP_PUSH <32-bytes of data> OP_EQUALVERIFY OP_CHECKSIG[VERIFY]`
2. A topmost stack element before an execution of Bitcoin script matching the pattern 
   `OP_PUSH <32- or 33-bytes of data> [OP_EQUALVERIFY] OP_CHECKSIG[VERIFY]`, when the previously executed command
   was not `OP_HASH160`
3. Each of the `n` topmost 33-byte-long stack elements before an execution of Bitcoin script matching the pattern 
   `OP_PUSH <n> OP_CHECKMULTISIG[VERIFY]`
4. An OP_RETURN code followed by 34 bytes, with the first two bytes set to `0xFFFF` means that the following 32 bytes
   represent a tagged hash (according to the procedure described in [12]) of a public key.

It follows, that the CC for non-P2(W)PK(H) and non-P2S outputs can be verified only if the whole source script is 
presented. In case of usage of Taproot, MAST or other technology hiding some parts of the script, a whole source script 
MUST BE presented to a verifier in order to verify the actual cryptographic commitment.

The CC verification requires all parts of scripts (lockScript, witnessScript, sigScript etc) to be assembled together,
before the patter-matching with the above given instructions can be performed.


### Deterministic cryptographic commitments to public key fingerprints within transaction output

For all public keys fingerprints defined according to the procedure described above a cryptographic commitment procedure
via key tweaking (as defined in LNPBPS-1 [12]) MUST BE applied.

To verify the commitment, the original public keys (before the tweaking procedure applied) and the source data for which
the commitment is constructed for MUST BE presented to the verifier. The verifier to make sure that the commitment is
valid MUST apply the algorithm from LNPBPS-1 [12] to each of the public keys.


## Compatibility

The proposed cryptographic commitment scheme is fully compatible with all previously-existed commitments for P2PK,
P2PKH and P2WPH transaction outputs, since they always contain only a single public key.

The standard is not compliant with previously used OP_RETURN-based cryptographic commitments, like OpenTimestamps [1],
since it utilises a hash of the tweaked public key for the OP_RETURN push data, and not the the actual commitment value
(cryptographic digest of the message to which the party commits to). In order to avoid any ambiguity, we are proposing
first, to use with OP_RETURN data a protocol-tagged hash, according to LNPBPS-1 [12], and also to add a special two-byte
prefix `0xFFFF` to render total OP_RETURN data size incompatible with either 32-bytes hashes or 33/32-bytes serialized
public keys, and also easily distinguishable from other protocols using OP_RETURN, to the best author's knowledge.

The author does not aware of any P2(W)SH or non-OP_RETURN P2S cryptographic commitment schemes existing before this
proposal, and it is highly probable that the standard is not compatible with ones if they were existing.

The proposed standard is compliant with current Taproot proposal [14], since it requires exposure of the complete
script with all it branches, allowing verification that all public keys participating in the script were the part of
the commitment procedure.


## Rationale

### Unification with OP_RETURN-based commitments

While it is possible to put a deterministic CC commitments into OP_RETURN-based outputs like with [1], their format
was modified for a unification purposes with the rest of the standard. This will help to reduce the verification and
commitment code branching, preventing potential bugs in the implementations.

### Continuing support for P2PK outputs

While P2PK outputs are considered obsolete and are vulnerable to a potential quantum computing attacks, it was decided
to include them into the specification for unification purposes.

### Committing to all public keys present in the script

With partial commitments, having some of the public keys tweaked with the message digest, and some left intact, it will
be impossible to define a simple and deterministic commitment scheme for an arbitrary script and output type that will
prevent any potential double-commitments.


## Reference implementation

<https://github.com/LNP-BP/rust-lnpbp/blob/master/src/commitments/script.rs>


## Acknowledgements

Authors would like to thank:
* Giacomo Zucco and Alekos Filini for their initial work on the commitment schemes as a part of early RGB effort [13]; 
* Christian Decker for pointing out on Lightning Network incompatibility with all existing cryptographic commitment 
  schemes;


## References

1. Peter Todd. OpenTimestamps: Scalable, Trust-Minimized, Distributed Timestamping with Bitcoin. 
   <https://petertodd.org/2016/opentimestamps-announcement>
2. Peter Todd. Preventing Consensus Fraud with Commitments and Single-Use-Seals.
   <https://petertodd.org/2016/commitments-and-single-use-seals>
3. [Eternity Wall's "sign-to-contract" article](https://blog.eternitywall.com/2018/04/13/sign-to-contract/)
4. Adam Back, Matt Corallo, Luke Dashjr, et al. Enabling Blockchain Innovations with Pegged Sidechains (commit5620e43).
   Appenxix A. <https://blockstream.com/sidechains.pdf>.
5. <https://exonum.com/doc/version/latest/advanced/bitcoin-anchoring/>
6. Pieter Wuille. Taproot proposal. <https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2019-May/016914.html>
7. Gregory Maxwell. Taproot: Privacy preserving switchable scripting.
   <https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2018-January/015614.html>
8. Gregory Maxwell. Graftroot: Private and efficient surrogate scripts under the taproot assumption.
   <https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2018-February/015700.html>
9. Andrew Poelstra. Scriptless scripts. <http://diyhpl.us/wiki/transcripts/layer2-summit/2018/scriptless-scripts/>
10. Lightning Network BOLT-3 standard, version 1.0. Sections ["Offered HTLC Outputs"]
    <https://github.com/lightningnetwork/lightning-rfc/blob/v1.0/03-transactions.md#offered-htlc-outputs>
    and ["Received HTLC Outputs"]
    <https://github.com/lightningnetwork/lightning-rfc/blob/v1.0/03-transactions.md#received-htlc-outputs>.
11. Rusty Russel. [Lightning-RFC (BOLTs) pull request #513]
    <https://github.com/lightningnetwork/lightning-rfc/pull/513>
12. Maxim Orlovsky. Key tweaking: collision-resistant elliptic curve-based commitments (LNPBP-1 Standard). 
    <https://github.com/LNP-BP/lnpbps/blob/master/lnpbps-0001.md>
13. RGB Protocol Specification, version 0.4. "Commitment Scheme" section.
    <https://github.com/rgb-org/spec/blob/old-master/01-rgb.md#commitment-scheme>
14. Pieter Wuille. Taproot: SegWit version 1 output spending rules (BIP standard proposal).
    <https://github.com/sipa/bips/blob/bip-schnorr/bip-taproot.mediawiki>


## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.


## Test vectors

TBD