```
LNPBP: 0002
Layer: Transactions (1)
Vertical: Bitcoin protocol
Title: Deterministic embedding of cryptographic commitments into transaction 
       output scriptPubkey
Authors: Dr Maxim Orlovsky <orlovsky@protonmail.ch>,
         Giacomo Zucco,
         Martino Salvetti,
         Federico Tenga,
         Sosthène
Comments-URI: https://github.com/LNP-BP/lnpbps/issues/4
Status: Proposal
Type: Standards Track
Created: 2019-10-27
License: CC0-1.0
```

## TOC

- [Abstract](#abstract)
- [Background](#background)
- [Motivation](#motivation)
- [Design](#design)
- [Specification](#specification)
  * [Commitment procedure](#commitment-procedure)
  * [Reveal procedure](#reveal-procedure)
  * [Verification procedure](#verification-procedure)
  * [Deterministic public key extraction from Bitcoin script](#deterministic-public-key-extraction-from-Bitcoin-script)
- [Compatibility](#compatibility)
- [Rationale](#rationale)
- [Reference implementation](#reference-implementation)
- [Acknowledgements](#acknowledgements)
- [References](#references)
- [License](#license)
- [Appendix A. Test vectors](#test-vectors)
  * [Correct test vectors](#correct-test-vectors)
  * [Invalid test vectors](#invalid-test-vectors)
  * [Protocol failures](#protocol-failures)


## Abstract

The standard defines an algorithm for deterministic embedding and verification
of cryptographic commitments based on elliptic-curve public key tweaking 
procedure defined in [LNPBP-1 standard](https://github.com/LNP-BP/LNPBPs/blob/master/lnpbp-0001.md)
inside `scriptPubkey` script for existing types of Bitcoin transaction output.


## Background

Cryptographic commitments embedded into bitcoin transactions is a widely-used
practice. Its application include timestamping [1], single-use seals [2],
pay-to-contract settlement schemes [3], sidechains [4], blockchain anchoring
[5], Taproot, Graftroot proposals [6, 7, 8], Scriptless scripts [9] and many
others.


## Motivation

A number of cryptographic commitment (CC) use cases require the commitment to be
present within non-P2(W)PK(H) outputs,which may contain multiple public keys and
need a clear definition of how the public key that contain the commitment can be
found within the bitcoin script itself. For instance, embedding CC into
Lightning Network (LN) payment channel state updates in the current version
requires the modification of an offered HTLC and received HTLC transaction
outputs[10], and these transactions contain only a single P2WSH output. With the
use of `option_anchor_outputs` [11] all outputs in the LN commitment transaction
becomes P2WSH outputs, also leading to a requirement for a standard and secure
way of making CC inside P2(W)SH outputs.

At the same time, P2(W)SH and other non-standard outputs (like explicit bare
script outputs, including OP_RETURN type) are not trivial to use for CC, since
CC requires deterministic definition of the actual commitment case. Normally,
one of the most secure and standard CC scheme uses homomorphic properties of the
public keys [12]; however multiple public keys may be used within non-P2(W)PK(H)
output. Generally, these outputs present a hash of the actual script, hiding the
used public keys or their hashes. Moreover, it raises the question of how the
public key within Bitcoin Script may be defined/detected, since it is possible
to represent the public key in a number of different ways within bitcoin script
itself (explicitly or by a hash, and it's not trivial to understand where some
hash stands for a public key or other type of preimage data). All this raises a
requirement to define a standard way and some best practices for CC in non-P2(W)
PK(H) outputs. This proposal tries to address the issue by proposing a common
standard on the use of public-key based CC [12] within all possible transaction
output types.

## Design

The protocol requires that exactly one public key of all keys present or
referenced in `scriptPubkey` and `redeemScript` must contain the commitment
(made with LNPBP-1 procedure) to a given message. This commitment is
deterministically singular, i.e. it can be proven that there is no other
alternative message that the given transaction output commits to under this
protocol. The singularity is achieved by committing to the sum of all original
(i.e. before the message commitment procedure) public keys controlling the
spending of a given transaction output. Thus, the given protocol covers all
possible options for `scriptPubkey` transaction output types.

The commitment consists of the updated `scriptPubkey` value, which may be embed 
into the transaction output, and an **extra-transaction proof** (ETP), required 
for the verification of the commitment. The structure and information in the
proof depends on the actual `scriptPubkey` type.

The protocol includes an algorithm for deterministic extraction of public keys
from a given Bitcoin script, based on Miniscript [16].


## Specification

### Commitment procedure

The **committing party**, having a message `msg`, must:
1. Collect all public keys instances (named **original public key set**) 
   related to the given transaction output, defined as:
   - a single public key for P2PK, P2PKH, P2WPK, P2WPK/WSH-in-P2SH type 
     of outputs;
   - If `OP_RETURN` `scriptPubkey` is used, it must contain a single public key 
     serialized in BIP-340 xcoordonly form right after `OP_RETURN` opcode; for 
     all other forms of `OP_RETURN` data algorithm must fail;
   - an internal public key for a Taproot output (P2TR);
   - all public keys from a `redeemScript` (for P2(W)SH and P2WSH-in-P2SH, 
     which in case of SegWit is contained within `witnessScript`) or 
     `scriptPubkey` (for custom bare script outputs), extracted according to the 
     [algorithm](#deterministic-public-key-extraction-from-bitcoin-script).  
   - In case of witness version > 1 the procedure must fail.
   Let's call this set of _n_ keys `P`.
2. Select a single public key `Po` from the set of the original public keys, 
   which will contain the commitment. It is advised that the corresponding 
   private key being controlled by the committing party, which will simplify 
   future spending of the output.
3. Run [LNPBP-1 commitment procedure](https://github.com/LNP-BP/LNPBPs/blob/master/lnpbp-0001.md#commitment-procedure) 
   on message `msg`, the set of original public keys `P`, the selected public 
   key `Po` and a protocol-specific `tag`, provided by the upstream protocol 
   using this standard. The procedure returns a tweaked public key `T`.
4. Construct necessary scripts and generate `scriptPubkey` of the required 
   type. If OP_RETURN `scriptPubkey` format is used, it MUST be serialized 
   according to the following rules:
   - only a single `OP_RETURN` code MUST be present in the `scriptPubkey` and it
     MUST be the first byte of it;
   - it must be followed by 32-byte push of the public key value `P` from the 
     step 2 of the algorithm, serialized according to the rules from [15];
   - if the tweaked public key serialized with bitcoin consensus serialization
     rules does not start with 0x02 as it's first (least significant) byte, the
     procedure must fail; or it may be repeated with a new public key picked at
     step 1.
5. Construct and store an **extra-transaction proof** (ETP), which structure 
   depends on the generated `scriptPubkey` type:
   a) value of `Po`, corresponding to:
      * the original intermediary public key for V1 witness Taproot output;
      * single original public key used in the tweak procedure before the tweak
        was applied;
   b) deterministic script reconstruction data, i.e.:
      * untweaked `redeemScript` (for P2SH outputs), or `witnessScript` 
        (for P2WSH SegWit v0 native or P2WSH-in-P2SH SegWit legacy outputs), 
        constructed using set of the *original public keys*;
      * the "taptweak" hash (i.e. the tagged hash of the Merkle root of the 
        TapScript branches) for v1 witness output (Taproot);
      * for other types of outputs no other data are required for the ETP.


### Reveal procedure

The **reveal protocol** is usually run between the committing and verifying 
parties; however it may be used by the committing party to make the proofs 
of the commitment public. These proofs include:
* `scriptPubkey` from the transaction output containing the commitment;
* *original message* `msg` to which the *comitting party* has committed;
* *extra-transaction proof* (ETP), constructed at the 5th step of the 
  [commitment protocol](#commitment);
* (optional) proofs that the `scriptPubkey` is a part of a transaction 
  included into the bitcoin chain containing the largest known amount of work at 
  depth satisfying a *verifying party* security policy (these proofs may be
  reconstructed/verified by the verifying party itself using its trusted Bitcoin 
  Core server);


### Verification procedure

The verification process runs by repeating steps of the commitment protocol
using the information provided during the *reveal phase* and verifying the
results for their internal consistency; specifically:
1. Original public key set reconstruction:
   - if *extra-transaction proof* (ETP) provides script data (pt. 5.b from the
     commitment procedure) parse it according to deterministic key collection
     [algorithm](#deterministic-public-key-extraction-from-bitcoin-script)
     and collect all the original public keys from it; fail the verification
     procedure if parsing fails;
   - otherwise use an original public key provided as a part of ETP as a single-
     element set
2. Run LNPBP-1 verification procedure repeating step 3 from the commitment
   procedure, using original public key value from the *extra-transaction proof*
3. Construct `scriptPubkey'`, matching the type of the `scriptPubkey` in the 
   transaction and matching *extra-transaction proof* data using the tweaked 
   version of the public key in the same way as was perfomed at step 4 of the
   commitment procedure. If there can be multiple matching `scriptPubkey` types
   for a given data, construct a variant for each of them.
4. Make sure that one of the `scriptPubkey'` values generated at the previous
   step, matches byte by byte the `scriptPubkey` provided during the *reveal
   phase*; otherwise fail the verification.


### Deterministic public key extraction from Bitcoin Script

1. The provided script MUST be parsed with Miniscript [16] parser; if the parser
   fails the procedure MUST fail.
2. Iterate over all branches of the abstract syntax tree generated by the
   Miniscript parser, running the following algorithm for each node:
   - if a public key hash is met (`pk_h` Miniscript command) and it can't be 
     resolved against known public keys or other public keys extracted from the
     script, fail the procedure;
   - if a public key is found (`pk`) add it to the list of the collected public 
     keys;
   - for all other types of Miniscript commands iterate over their branches.
3. Select unique public keys (i.e. if some public key is repeated in different 
   parts of the script/in different script branches, pick a single instance of 
   it). Compressed and uncompressed versions of the same public key must be 
   treated as the same public key under this procedure.
4. If no public keys were found fail the procedure; return the collected keys 
   otherwise.

By "miniscript" we mean usage of `rust-miniscript` library v2.0.0 (commit 
`463fc1eadac2b46de1cd5ae93e8255a2ab34b906`) which may be found at 
<https://github.com/LNP-BP/rust-miniscript/commit/463fc1eadac2b46de1cd5ae93e8255a2ab34b906>


## Compatibility

The proposed cryptographic commitment scheme is fully compatible, and works with
LNPBP-1 standard [12] for constructing cryptographic commitments with a set of 
public keys.

The standard is not compliant with previously used OP_RETURN-based cryptographic
commitments, like OpenTimestamps [1], since it utilises plain value of the
public key with the commitment for the OP_RETURN push data.

The author is not aware of any P2(W)SH or non-OP_RETURN cryptographic commitment
schemes existing before this proposal, and it is highly probable that the
standard is not compatible with ones if they were existing.

The proposed standard is compliant with current Taproot proposal [14], since it
can use intermediate Taproot key to store the commitment.

Standard also should be interoperable with Schnorr's signature proposal [15],
since it uses miniscript supporting detection of public keys serialized with new
BIP-340 rules.


## Rationale

### Continuing support for P2PK outputs

While P2PK outputs are considered obsolete and are vulnerable to a potential
quantum computing attacks, it was decided to include them into the specification
for unification purposes.

### Support OP_RETURN type of outputs

OP_RETURN was originally designed to reduce the size of memory-kept UTXO data;
however it is not recommended to use it since it bloats blockchain space usage.
This protocol provides support for this type of outputs only because some
hardware (like HSM modules or hardware wallets) can't tweak public keys inside
outputs and produce a correct signature after for their spending, and keeping
the commitment in OP_RETURN's can be an only option. For all other cases it is
highly recommended not to use OP_RETURN outputs for the commitments and use
tweaking of other types of outputs instead.

### Unification with OP_RETURN-based commitments

While it is possible to put a deterministic CC commitments into OP_RETURN-based
outputs like with [1], their format was modified for unification purposes with
the rest of the standard. This will help to reduce the verification and
commitment code branching, preventing potential bugs in the implementations.

### Custom serialization of public keys in OP_RETURN

This saves one byte of data per output, which is important in case of blockchain
space.

While we could pick a new BIP340-based rules for public key serialization [15],
this will require software to support new versions of the secp256k1 libraries 
and language-specific wrappers, which may be a big issue for legacy software
usually used by hardware secure modules, which require presence of OP_RETURN's
(see rationale points above).

### Support for pre-SegWit outputs

These types of outputs are still widely used in the industry and it was decided
to continue their support, which may be dropped in a higher-level requirements
by protocols operating on top of LNPBP-2.

### Committing to the sum of all public keys present in the script

Having some of the public keys tweaked with the message digest, and some left
intact will make it impossible to define a simple and deterministic commitment
scheme for an arbitrary script and output type that will prevent any potential
double-commitments.

### Deterministic public key extraction for a Bitcoin script

We use determinism of the proposed Miniscript standard to parse the Bitcoin
script in an implementation-idependet way and in order to avoid the requirement
of the full-fledged Bitcoin Script interpreter for the commitment verification.

The procedure fails on any public key hash present in the script, which prevents
multiple commitment vulnerability, when different parties may be provided with
different *extra-transaction proof* (ETP) data, such that they will use
different value for the `S` public key matching two partial set of the public
keys (containing and not containing public keys for the hashed values).

The algorithm does not require that all of the public keys will be used in
signature verification for spending the given transaction output (i.e. public
keys may not be prefixed with `[v]c:` code in Miniscript [16]), so the
committing parties may have a special public key shared across them for
embedding the commitment, without requiring to know the corresponding private
key to spend the output.

### Use of Taproot intermediate public key

With Taproot we can't tweak the public key contained in the `scriptPubkey`
directly since it will invalidate its commitment to the Tapscript and to the
intermediate key, rendering output unspendable. Thus, we put tweak into the
underlying intermediate public key as the only avaliable option.


## Reference implementation

<https://github.com/LNP-BP/rust-lnpbp/blob/master/src/bp/dbc/lockscript.rs>


## Acknowledgements

Authors would like to thank: 
* Giacomo Zucco and Alekos Filini for their initial work on the commitment 
  schemes as a part of early RGB effort [13];
* Dr Christian Decker for pointing out on Lightning Network incompatibility with
  all existing cryptographic commitment schemes.


## References

1. Peter Todd. OpenTimestamps: Scalable, Trust-Minimized, Distributed 
   Timestamping with Bitcoin.
   <https://petertodd.org/2016/opentimestamps-announcement>
2. Peter Todd. Preventing Consensus Fraud with Commitments and Single-Use-Seals.
   <https://petertodd.org/2016/commitments-and-single-use-seals>
3. [Eternity Wall's "sign-to-contract" article](https://blog.eternitywall.com/2018/04/13/sign-to-contract/)
4. Adam Back, Matt Corallo, Luke Dashjr, et al. Enabling Blockchain Innovations 
   with Pegged Sidechains (commit5620e43). Appenxix A. 
   <https://blockstream.com/sidechains.pdf>.
5. <https://exonum.com/doc/version/latest/advanced/bitcoin-anchoring/>
6. Pieter Wuille. Taproot proposal. 
   <https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2019-May/016914.html>
7. Gregory Maxwell. Taproot: Privacy preserving switchable scripting.
   <https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2018-January/015614.html>
8. Gregory Maxwell. Graftroot: Private and efficient surrogate scripts under the
   taproot assumption.
   <https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2018-February/015700.html>
9. Andrew Poelstra. Scriptless scripts. 
   <http://diyhpl.us/wiki/transcripts/layer2-summit/2018/scriptless-scripts/>
10. Lightning Network BOLT-3 standard, version 1.0. Sections ["Offered HTLC 
    Outputs"]
    <https://github.com/lightningnetwork/lightning-rfc/blob/v1.0/03-transactions.md#offered-htlc-outputs>
    and ["Received HTLC Outputs"]
    <https://github.com/lightningnetwork/lightning-rfc/blob/v1.0/03-transactions.md#received-htlc-outputs>.
11. [BOLT-3 Lightning network specification](https://github.com/lightningnetwork/lightning-rfc/blob/master/03-transactions.md#to_remote-output)
12. Maxim Orlovsky, et al. Key tweaking: collision-resistant elliptic 
    curve-based commitments (LNPBP-1 Standard).
    <https://github.com/LNP-BP/lnpbps/blob/master/lnpbp-0001.md>
13. RGB Protocol Specification, version 0.4. "Commitment Scheme" section.
    <https://github.com/rgb-org/spec/blob/old-master/01-rgb.md#commitment-scheme>
14. Pieter Wuille. Taproot: SegWit version 1 spending rules.
    <https://github.com/bitcoin/bips/blob/master/bip-0341.mediawiki>
15. Pieter Wuille. Schnorr Signatures for secp256k1.
    <https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki>
16. Pieter Wuille, Andrew Poelsta. Miniscript. 
    <http://bitcoin.sipa.be/miniscript/>

## License

This document is licensed under the Creative Commons CC0 1.0 Universal license.


## Appendix A. Test vectors

### 1. Correct test vectors

TBD

### 2. Invalid test vectors

TBD

### 3. Edge cases: protocol failures

TBD
