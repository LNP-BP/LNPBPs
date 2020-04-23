```
LNPBP: 0002
Layer: Transactions (1)
Field: Cryptographic commitments
Title: Deterministic embedding of LNPBP1-type commitments into `scriptPubkey` of a transaction output
Authors: Dr Maxim Orlovsky <orlovsky@protonmail.ch>,
         Giacomo Zucco,
         Alekos Filini,
         Martino Salveti,
         Federico Tenga
Comments-URI: https://github.com/LNP-BP/lnpbps/issues/4
Status: Draft
Type: Standards Track
Created: 2019-27-10
License: CC0-1.0
```

## Abstract

The standard defines an algorithm for deterministic embedding and verification of cryptographic commitments based
on elliptic-curve public and private key modifications (tweaks) inside the existing types of Bitcoin transaction output
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


## Design

The protocol requires that exactly one public key of all keys present or referenced in `scriptPubkey` and `redeemScript`
must contain the commitment (made with LNPBP-1 procedure) to a given message. This commitment is deterministically
singular, i.e. it can be proven that there is no other alternative message that the given transaction output commits
to under this protocol. The singularity is achieved by committing to the sum of all original (i.e. before the message
commitment procedure) public keys controlling the spending of a given transaction output. Thus, the given protocol
modifies LNPBP-1 standard [12] for the case of multiple public keys and covers all possible options for `scriptPubkey`
transaction output types.

The commitment consists of the new version of `scriptPubkey`, which may be embed into the transaction output, and an
extra-transaction proof (ETP), required for the verification of the commitment. The structure and information in the
proof depends on the used `scriptPubkey` type.

The protocol includes an algorithm for deterministic extraction of public keys from a given Bitcoin script, based on
Miniscript [16].


## Specification

### Commitment

The **committing party**, having a message `msg`
1. Collects all public keys instances (named **original public keys**) related to the given transaction output under this standard,
   namely:
   - a single public key for P2PK, P2PKH, P2WPK, P2WPK-in-P2SH type of outputs, `redeemScripts` or custom non-P2(W)SH
     `scriptPubkey` requiring a single signature;
   - an arbitrary public key (even with an unknown corresponding private key), if `OP_RETURN` `scriptPubkey` is used;
   - an intermediate public key for the Taproot;
   - all signing public keys from a `redeemScript` or custom non-P2(W)SH `scriptPubkey`, defined according to the
     [algorithm](#deterministic-public-key-extraction-from-bitcoin-script).
2. Computes en elliptic curve point and its corresponding public key `S` as sum of elliptic curve points corresponding
   to the original public keys.
3. Selects a single public key `P` from the set of the original public keys, which will contain the commitment.
4. Runs LNPBP-1 commitment protocol on message `msg`, the selected public key `P` and public key `S` with the following
   modification:
   - on the second step of the protocol (according to LNPBP-1 specification [12]) HMAC-SHA256 is provided with the value
     of `S` (instead of `P`), i.e. hence we commit to the sum of all the original public keys;
   - on the forth step of the protocol the tweaking-factor based point `F` is added to the `P` (a single original public
     key), resulting in key `T`;
   - if the number of original public keys defined at step 1 exceeds one, `LNPBP2` tag MUST BE used instead of `LNPBP1`;
   - a protocol-specific tag is provided by the upstream protocol using this standard.
5. Constructs necessary scripts and generates `scriptPubkey` of the required type. If OP_RETURN `scriptPubkey` format is
   used, it MUST be serialized according to the following rules:
   - only a single `OP_RETURN` code MUST be present in the `scriptPubkey` and it MUST be the first byte of it;
   - it must be followed by 32-byte push of the public key value `P` from the step 2 of the algorithm, serialized
     according to from [15]; if the resulting public key containing the commitment is non-square, a new public key MUST
     be picked and procedure from step 4 MUST BE repeated once more.
6. Constructs and stores an **extra-transaction proof** (ETP), which structure depends on the generated `scriptPubkey`
   type:
   a) value of `S`, corresponding to:
      * single original public key for P2PK, P2PKH, P2WPK, P2WPK-in-P2SH and OP_RETURN-type of outputs;
      * the original intermediary public key for P2TR (V1 witness Taproot output);
      * sum of all public keys required to spend P2SH, P2WSH or P2WSH-in-P2SH output;
   b) deterministic script reconstruction data, i.e.:
      * untweaked `redeemScript` for P2SH, P2WSH or P2WSH-in-P2SH `scriptPubkey`, constructed using the *original
        public keys*, which MUST be ordered within the script in exactly the same way they are ordered in the actual
        `redeemScript` containing the public key with the commitment;
      * the "taptweak" hash (i.e. the tagged hash of the Merkle root of the TapScript branches) for P2TR (V1 witness
        output);
      * for other types of outputs no data are provided in this section of ETP.


### Reveal

The **revel protocol** is usually run between the committing and verifying parties; however it may be used by the
committing party to publicaly revel the proofs of the commitment. These proofs include:
* `scriptPubkey` from the transaction output containing the commitment;
* *original message* to which the *comitting party* has committed;
* *extra-transaction proof* (ETP), constructed at the 6th step of the [commitment protocol](#commitment);
* (optional) proofs that the `scriptPubkey` is a part of the transaction included into the bitcoin chain containing
  the largest known amount of work at depth satisfying a *verifying party* security policy (these proofs may be
  reconstructed/verified by the verifying party itself using its trusted Bitcoin Core server);


### Verify

The verification process runs by repeating steps of the commitment protocol using the information provided during the
*revel phase* and verifying the results for their internal consistency; specifically:
1. Public key consistency verification:
   - collect all the original public keys from the deterministic script reconstruction data of the *extra-transaction
     proof* (ETP), if present; and
   - check that the sum of these public keys `Z'` equals to the value of `S` from the ETP, otherwise fail the
     verification.
2. For each of the original public keys (or `S`, if there were no other original public keys provided in ETP):
   - construct a commitment to the `msg` according to the commitment procedure described at the step 4 of the
     [commitment protocol](#commitment);
   - construct `scriptPubkey'`, matching the type of the `scriptPubkey` provided during the *revel phase*, using the
     tweaked version of this public key and ETP data for deterministic script reconstruction.
3. Make sure that one, and only one value of `scriptPubkey'` generated at previous step, byte by byte matches
   the `scriptPubkey` provided during the *revel phase*; otherwise (if no matches found or multiple matches are present)
   fail the verification.


### Deterministic public key extraction from Bitcoin Script

1. The provided script MUST be parsed with Miniscript [16] parser; if the parser fails the procedure MUST fail.
2. Iterate over all branches of the abstract syntax tree generated by the Miniscript parser, running the following
   algorithm for each node:
   - if a public key hash is met (`pk_h` Miniscript command) and it can't be resolved against known public keys or other public keys extracted from the script, fail the procedure;
   - if a public key is found (`pk`) add it to the list of the collected public keys;
   - for all other types of Miniscript commands iterate over their branches.
3. Select unique public keys (i.e. if some public key is repeated in different parts of the script/in different
   script branches, pick a single instance of it). Compressed and uncompressed versions of the same public key must be treaded as the same public key under this procedure.
4. If no public keys were found fail the procedure; return the collected keys otherwise.

**NB: SUBJECT TO CHANGE UPON RELEASE**
By "miniscript" we mean usage of `rust-miniscript` library at commit `a5ba1219feb8b5a289c8f12176d632635eb8a959`
which may be found on <https://github.com/LNP-BP/rust-miniscript/commit/a5ba1219feb8b5a289c8f12176d632635eb8a959>


## Compatibility

The proposed cryptographic commitment scheme is fully compatible with any other LNPBP1-based commitments for the
case of P2PK, P2PKH and P2WPH transaction outputs, since they always contain only a single public key and the original
`LNPBP1` tag is preserved for the commitment procedure in this case.

The standard is not compliant with previously used OP_RETURN-based cryptographic commitments, like OpenTimestamps [1],
since it utilises plain value of the public key with the commitment for the OP_RETURN push data.

The author does not aware of any P2(W)SH or non-OP_RETURN P2S cryptographic commitment schemes existing before this
proposal, and it is highly probable that the standard is not compatible with ones if they were existing.

The proposed standard is compliant with current Taproot proposal [14], since it requires exposure of the complete
script with all it branches, allowing verification that all public keys participating in the script were the part of
the commitment procedure.

TODO: Schnorr compatibility

## Rationale

### Unification with OP_RETURN-based commitments

While it is possible to put a deterministic CC commitments into OP_RETURN-based outputs like with [1], their format
was modified for a unification purposes with the rest of the standard. This will help to reduce the verification and
commitment code branching, preventing potential bugs in the implementations.

### Continuing support for P2PK outputs

While P2PK outputs are considered obsolete and are vulnerable to a potential quantum computing attacks, it was decided
to include them into the specification for unification purposes.

### Committing to the sum all public keys present in the script

With partial commitments, having some of the public keys tweaked with the message digest, and some left intact, it will
be impossible to define a simple and deterministic commitment scheme for an arbitrary script and output type that will
prevent any potential double-commitments.

### Deterministic public key extraction for a Bitcoin script

We use determinism of the proposed Miniscript standard to parse the Bitcoin script in an implementation-idependet way
and in order to avoid the requirement of the full-fledged Bitcoin Script interpreter for the commitment verification.

The procedure fails on any public key hash present in the script, which prevents multiple commitment vulnerability,
when different parties may be provided with different *extra-transaction proof* (ETP) data, such that they will
use different value for the `S` public key matching two partial set of the public keys (containing and not containing
public keys for the hashed values).

The algorithm does not require that all of the public keys will be used in signature verification for spending the
given transaction output (i.e. public keys may not be prefixed with `[v]c:` code in Miniscript [16]), so the committing
parties may have a special public key shared across them for embedding the commitment, without requiring to know the
corresponding private key to spend the output.

### Use of the intermediate public key for Taproot


### Support of non-SegWit outputs


## Reference implementation

<https://github.com/LNP-BP/rust-lnpbp/blob/master/src/cmt/txout.rs>


## Acknowledgements

Authors would like to thank:
* Giacomo Zucco and Alekos Filini for their initial work on the commitment schemes as a part of early RGB effort [13];
* Dr Christian Decker for pointing out on Lightning Network incompatibility with all existing cryptographic commitment
  schemes.


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
12. Maxim Orlovsky, et al. Key tweaking: collision-resistant elliptic curve-based commitments (LNPBP-1 Standard).
    <https://github.com/LNP-BP/lnpbps/blob/master/lnpbp-0001.md>
13. RGB Protocol Specification, version 0.4. "Commitment Scheme" section.
    <https://github.com/rgb-org/spec/blob/old-master/01-rgb.md#commitment-scheme>
14. Pieter Wuille. Taproot: SegWit version 1 output spending rules (BIP standard proposal).
    <https://github.com/sipa/bips/blob/bip-schnorr/bip-taproot.mediawiki>
15. Pieter Wuille. Schnorr Signatures for secp256k1.
    <https://github.com/sipa/bips/blob/bip-schnorr/bip-schnorr.mediawiki>
16. Pieter Wuille, Andrew Poelsta. Miniscript. <http://bitcoin.sipa.be/miniscript/>

## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.


## Test vectors

TBD
