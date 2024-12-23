```
LNPBP: 0046
Vertical: Lightning network protocol
Title: Deterministic derivation paths for LNP
Author: Dr Maxim Orlovsky <orlovsky@protonmail.ch>,
Comments-URI: https://github.com/LNP-BP/LNPBPs/pull/120
Status: Draft
Type: Standards Track
Created: 2021-12-07
Finalized: not yet
Based on: BIP-32, BIP-43, BOLT-3
License: CC0-1.0
```

- [Abstract](#abstract)
- [Background](#background)
- [Motivation](#motivation)
- [Design](#design)
- [Specification](#specification)
- [Compatibility](#compatibility)
  - [c-Lightning](#c-lightning)
  - [LND](#lnd)
  - [Eclair](#eclair)
  - [LNP Node](#lnp-node)
  - [Electrum](#electrum)
- [Rationale](#rationale)
  - [Choice of `9735h` for BIP-43 purpose](#choice-of-9735h-for-bip-43-purpose)
  - [Use of channel ids for channel basepoint derivation](#use-of-channel-ids-for-channel-basepoint-derivation)
  - [Selection of bits for channel basepoint](#selection-of-bits-for-channel-basepoint)
  - [Use of unhardeded derivation between basepoints](#use-of-unhardeded-derivation-between-basepoints)
  - [Shutdown key derived from funding wallet](#shutdown-key-derived-from-funding-wallet)
  - [Using unhardened indexes for per-commitment points](#using-unhardened-indexes-for-per-commitment-points)
- [Reference implementation](#reference-implementation)
- [Acknowledgements](#acknowledgements)
- [References](#references)
- [Copyright](#copyright)
- [Test vectors](#test-vectors)


## Abstract

The proposal standardizes derivation paths used by lightning nodes, for both
BOLT-compatible and Bifrost networks.


## Background

BOLT-3 define a number of "basepoints" - public and private keys used
in channel transaction construction and the rules for derivation of the 
specific keys for each of the transactions out of the basepoint set.

This proposal fills in the gap of how these basepoints may be deterministically
derived from a single extended key.


## Motivation

A standard for deterministic derivation of keys used in channel transaction
construction will allow users to always recover the whole information out of a 
seed phrase independently from a specific node implementation used for channel 
creation.


## Design

We define a **extended lightning key** as an extended master private key used for
all derivations of channel basepoints, funding wallet and **node key**. This key is
derivable from the extended master key / seed phrase using a specific derivation
path purpose value `9735'`.


## Specification

**Extended lightning key** should be derived from a extended master key using 
the following derivation path:

`m/9735'/`

where `9735'` is a BIP-43 purpose field reserved for this standard.

**Node key** is derived from the *extended lightning key* using 
`/chain'/0'/node_index'` derivation, where `chain'` is a hardened index 
specifying blockchain operated by the node, and `node_index'` is a hardened
incremental index for the node among all nodes managed by a seed owner.

**Channel basepoint** is derived from the *extended lightning key* using
`/chain'/1'/ln_ver'/channel'` derivation, where:
- `channel'` is a hardened index constructed from the *initial channel id* 
  most significant bits starting from 1 to 32 using zero-based indexing. 
  For BOLT-defined lightning channels *initial channel id* is the temporary 
  channel id value; for Bifrost channel it is the channel id value.
- `ln_ver'` is a hardened index set to `0'` for BOLT-defined lightning channels
  and `1'` for Bifrost channels.

Derivation paths for the base points are the following:

Basepoint name | Derivation suffix | Full derivation path
-------------- | -------------------------------------------
Funding        | `/0`              | `m/9735'/chain'/1'/ln_ver'/channel'/0`
Payment        | `/1`              | `m/9735'/chain'/1'/ln_ver'/channel'/1`
Delayed        | `/2`              | `m/9735'/chain'/1'/ln_ver'/channel'/2`
Revocation     | `/3`              | `m/9735'/chain'/1'/ln_ver'/channel'/3`
Per-commitment | `/4/*`            | `m/9735'/chain'/1'/ln_ver'/channel'/4/0`
HTLC           | `/5`              | `m/9735'/chain'/1'/ln_ver'/channel'/5`
PTLC           | `/6`              | `m/9735'/chain'/1'/ln_ver'/channel'/6`

**Funding wallet** used for keeping funds by a lightning node for 
constructing funding transactions is derived with 
`/chain'/2'/case/index` path, where `case` is an equivalent of `change` field
with RGB extensions (see ___) and `index` is a sequential index number.

Example of extended keys used by lightning node #0 on bitcoin network
for a BOLT-defined channel with temporary channel id 
`2938ce5c0cae4b2af072065dc7dcea68de5e3de0c936742e27e045c8650c6a79`:

Extended keys            | Derivation
------------------------ | -----------
Lightning key            | `m/9735'/`
Node key                 | `m/9735'/0'/0'/0'`
Channel basepoint        | `m/9735'/0'/1'/0'/691588700'` (index equals to `0x2938ce5c`)
Per-commitment points    | `m/9735'/0'/1'/0'/691588700'/4/<per_commitment_no>`
Funding wallet           | `m/9735'/0'/2'/0/*`
Funding wallet change    | `m/9735'/0'/2'/1/*`
Shutdown                 | `m/9735'/0'/2'/2/*`
Funding wallet RGB20     | `m/9735'/0'/2'/200/*`
RGB20 change             | `m/9735'/0'/2'/201/*`
Shutdown RGB20           | `m/9735'/0'/2'/202/*`

Please note that the channel shutdown key is derived from a funding wallet -
and not from a channel basepoint.


## Compatibility

TBD: investigate derivation paths used by specific lightning nodes

### c-Lightning

### LND

### Eclair

### LNP Node

LNP Node is fully compatible with the standard

### Electrum


## Rationale

### Choice of `9735h` for BIP-43 purpose

BIP-43 defines that new purposes must be created with new BIPs and their 
hardened index representations must match that BIP number. Since we this
standard is created before BIP proposal for a new purpose, we need to
choose some large number which will not be used by BIPs in a foreseable
future. `9735` is the unicode character value for lightning (☇) also used
as a port number in lightning network.

### Use of channel ids for channel basepoint derivation

Channels may be created asynchronously and it is hard to get sequential
numbering within the lightning node. Also, if the node data are lost, it will 
be hard to guess which channel had which sequence index. Thus, the only way to
get a channel-specific derivation index is to use some of channel id bits.

We can't use channel funding transaction id since it will be known only upon
key derivation.

### Selection of bits for channel basepoint

We can use only 31 bits, since the most significant bit of derivation index is
occupied by hardering flag (see BIP-32). We start with the second most 
significant channel bit to make it easy visually compare index with channel id
without any bit shifts.

### Use of unhardeded derivation between basepoints

Each channel is derived with hardened derivation, which allows to separate
node from channels: if a node-level extended public – or even extended private
key is leaked, it still be impossible to guess from the key information which 
channels ere created or closed with that node. From the other hand, if one
needs to disclose the full information about the channel transaction it will
be sufficient to have a single extended public key for **channel basepoint**
to derive all other public keys any channel transaction.

### Shutdown key derived from funding wallet

Funding wallet may be a separate process - or separate cold wallet, isolated
from the rest of the lightning node. Deriving shutdown key from funding wallet
allows it to use funds from closed channels for funding new channels without
the need for additional interactivity with the lightning node.

### Using unhardened indexes for per-commitment points

This allows user to reconstruct channel history without the need to access channel
extended private key.


## Reference implementation

The reference implementation of this derivation standard is done in a
[LNP Core Library](https://github.com/LNP-BP/lnp-core) as a part of
`legacy::channel::Keyset`, `bifrost::channel::Keyset` and `NodeAccount`
structures.


## Acknowledgements


## References


## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.


## Test vectors

