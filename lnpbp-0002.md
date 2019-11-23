```
LNPBP: 0002
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
Taproot, Graftroot proposals [6, 7, 8], Scriptless scripts [9] and many others. 


## Motivation

A number of cryptographic commitment (CC) use cases require the commitment to be present within non-P2(W)PK(H) outputs,
which may contain multiple public keys and need a more clear definition of how the public key can be found within the
the bitcoin script itself. For instance, embedding CC into Lightning Network (LN) payment channel state updates in the 
current version require modification of an offered HTLC and received HTLC transaction outputs [10], and these 
transactions contain only a single P2WSH output. With the following updates [11] LN will likely will change a single 
P2WPKH (named `to_remote`) output within the commitment transaction, also leading to a requirement for a standard and 
secure way of making CC inside P2(W)SH outputs.

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

The **committing party** has:
1. To decide on the type of transaction output `scriptPubkey` that will contain the commitment.
2. To construct and keep a set `S` of the original public keys that will be used in the output construction according
   to the selected type.
3. To commit to the message and put the commitment into EACH of the public keys from the set `S` according to the
   selected procedure. We recommend using the procedure defined in LNPBP-1 [12] and choose the first reveal scheme.
5. Construct the transaction output and (if required) the associated `redeemScript` using the version of public keys
   containing commitment.

The **verifying party** SHOULD be provided with the transaction output and supplementary data, which structure depends on 
the structure of `scriptPubkey` of the transaction output, after which it has to perform the following actions:
1. Extract (for a verifying party) from the `scriptPubkey` field and provided supplementary data a set `S` of eligible 
   public keys for the cryptographic commitment with the following procedure, depending on the script type within 
   `scriptPubkey`:
    - **P2PK**: take a single public key `P` from the `scriptPubkey` itself; `S = { P }`
    - **P2PKH**: the supplementary data MUST provide a public key `O` matching the hash `h` within 
      `scriptPubkey`; then `S = { O: RIPEMD160(SHA256(O)) = h }`
    - **V0 witness, P2WPKH variant**: the supplementary data MUST provide the original public key `O` matching the hash
      `h` within the witness program inside the `scriptPubkey`; `S = { O: RIPEMD160(SHA256(O)) = h }`
    - **V0 witness, P2WSH variant**: the supplementary data MUST provide the set `S` plus a `redeemScript`
      (corresponding  to the last item inside the `witnessScript`) matching the script hash contained in the
      `scriptPubkey`. The set `S`  MUST be verified against the provided `redeemScript` according to
      [the procedure](#verification-of-public-keys-against-bitcoin-script) described below.
    - **V1 witness, P2TP** [**work in progress**: this part has to be revised following the acceptance of BIP-Taproot 
      proposal [14]]: the supplementary data MUST provide an intermediate public key `I` and Taproot tweak public key
      `T` such that `I + T` MUST equal to the public key serialized within the `scriptPubkey`. Then, `S = { I }`.
    - **P2SH**: the supplementary data MUST provide either:
        * for a ***V0 P2WPKH embedded into P2SH***, an original public key `O`, which, when serialized according to the 
          [rule from BIP141](https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki#p2wpkh-nested-in-bip16-p2sh) 
          first into a string `s = 0x160014<RIPEMD160(SHA256(O))>` and then hashed as `h' = RIPEMD160(SHA256(s))`. 
          The hash `h'` MUST match the hash `h` from the original P2SH: `h == h'`, and
          `S = { O: RIPEMD160(SHA256(0x160014<RIPEMD160(SHA256(O))>)) = h }`
        * for a ***V0 P2WSH embedded into P2SH***, the supplementary data MUST provide the set `S` plus a `redeemScript`, 
          corresponding to the last item in the `witnessScript`, which, when is serialized according to the 
          [rule from BIP141](https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki#p2wsh-nested-in-bip16-p2sh)
          first into a string `s = 0x220020<SHA256(SHA256(<redeemScript>))>` and then hashed as
          `h = RIPEMD160(SHA256((s))`. The hash `h'` MUST match the hash `h` from the original P2SH: `h == h'`. The set 
          `S` MUST be verified against the provided `redeemScript` according to
          [the procedure](#verification-of-public-keys-against-bitcoin-script) described below.
        * for a ***native P2SH***, the supplementary data MUST provide the set `S` plus a `redeemScript` matching the 
          script hash contained in the `scriptPubkey`. The set `S` MUST be verified against the provided `redeemScript`
          according to [the procedure](#verification-of-public-keys-against-bitcoin-script) described below.
    - **OP_RETURN**: the `scriptPubkey` MUST start with `OP_RETURN` code followed by a 32-byte push of the public key
      `R` serialized according to the BIP-Schnorr serialization rules [15], representing a sole member of the set `S`. 
    - **Other (non-standard) scripts**: the supplementary data MUST provide the set `S`, which `S` MUST be verified 
      against the provided `scriptPubkey` according to 
      [the procedure](#verification-of-public-keys-against-bitcoin-script) described below.

2. The supplementary data MUST contain for each of the public keys in the set `∀ P ∈ S` a corresponding original public 
   key `{ P' } = S'` and a message `msg` to which the commitment was created. Each of the public keys within the set `S` 
   MUST be verified to contain the commitment to the message `msg` basing on the original public key from `S'` according 
   to the selected commitment procedure.  We recommend using the procedure defined in LNPBP-1 [12] and choose the first 
   reveal scheme.


### Verification of public keys against Bitcoin Script

A verifier SHOULD be provided with the bitcoin script bytecode and a set of public keys `S`. The verifying party has to
make sure that the script does not contain any other public keys or their hashes, and that all public keys presented
in the script are present in the provided set `S`. In order to detect public keys and their hashes inside the bitcoin
script the verifier MUST follow the given procedure:
1. Start with an empty set of script-extracted public key hashes `H`
2. Scan the script for the presence of `OP_HASH160 OP_PUSH32 <32-bytes of data> OP_EQUALVERIFY OP_CHECKSIG[VERIFY]`
   template, extract the 32-byte sequences from the pattern and put each of them into the set `H`. Splice out each of
   the found script patterns from the script before the further processing.
3. Scan the script for the presence of `OP_PUSH32 <33-bytes of data> [OP_EQUALVERIFY] OP_CHECKSIG[VERIFY]`
   and `OP_PUSH <n> [<33-bytes of data>]+ OP_PUSH <m> OP_CHECKMULTISIG[VERIFY]`
   template, extract the 33-byte sequences from the pattern and make sure that they represent a valid x-coordinates of 
   Secp256k1 cure points. Compute a double SHA-256 hash of them and put each of the resulting values into the set `H`. 
   Splice out each of the found script patterns from the script before the further processing.
4. Scan the script for the presence of `OP_PUSH32 <65-bytes of data> [OP_EQUALVERIFY] OP_CHECKSIG[VERIFY]`
   and `OP_PUSH <n> [<65-bytes of data>]+ OP_PUSH <m> OP_CHECKMULTISIG[VERIFY]`
   template, extract the 65-byte sequences from the pattern and make sure that they represent a valid uncompressed
   public keys for the Secp256k1 curve points. Take their x-coordinates and compute a double SHA-256 hash of them and 
   put each of the resulting values into the set `H`. Splice out each of the found script patterns from the script 
   before the further processing.
5. Compute double SHA-256 hash of each of the public keys in set `S` and put it into the set `H'`.
6. Ensure that `H' = H`, i.e. they have the same number of elements and there is exactly one element from the set `H'`
   that is equal to an element from the set `H`. Otherwise, fail the verification procedure.


## Compatibility

The proposed cryptographic commitment scheme is fully compatible with all previously-existed commitments for P2PK,
P2PKH and P2WPH transaction outputs, since they always contain only a single public key.

The standard is not compliant with previously used OP_RETURN-based cryptographic commitments, like OpenTimestamps [1],
since it utilises a hash of the tweaked public key for the OP_RETURN push data, and not the the actual commitment value
(cryptographic digest of the message to which the party commits to). In order to avoid any ambiguity, we are proposing
first, to use with OP_RETURN data a protocol-tagged hash, according to LNPBP-1 [12], and also to add a special two-byte
prefix `0xFFFF` to render total OP_RETURN data size incompatible with either 32-bytes hashes or 33/32-bytes serialized
public keys, and also easily distinguishable from other protocols using OP_RETURN, to the best author's knowledge.

The author does not aware of any P2(W)SH or non-OP_RETURN P2S cryptographic commitment schemes existing before this
proposal, and it is highly probable that the standard is not compatible with ones if they were existing.

The proposed standard is compliant with current Taproot proposal [14], since it requires exposure of the complete
script with all it branches, allowing verification that all public keys participating in the script were the part of
the commitment procedure.

TODO: Schnorr compatibility, Taproot compatibility

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
    <https://github.com/LNP-BP/lnpbps/blob/master/lnpbp-0001.md>
13. RGB Protocol Specification, version 0.4. "Commitment Scheme" section.
    <https://github.com/rgb-org/spec/blob/old-master/01-rgb.md#commitment-scheme>
14. Pieter Wuille. Taproot: SegWit version 1 output spending rules (BIP standard proposal).
    <https://github.com/sipa/bips/blob/bip-schnorr/bip-taproot.mediawiki>
15. Pieter Wuille. Schnorr Signatures for secp256k1.
    <https://github.com/sipa/bips/blob/bip-schnorr/bip-schnorr.mediawiki>


## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.


## Test vectors

TBD