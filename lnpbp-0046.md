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
Based on: BIP-32, BOLT-3
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

We define a **extended node key** as an extended master private key used for
all derivations of both transaction-related keys and **node key**. This key is
derivable from the extended master key / seed phrase using a specific derivation
path.


## Specification

**Extended node key** should be derived from a extended master key using the
following derivation path:

`m/10046'/`

where `10046` is a BIP-43 purpose field reserved for this standard.

**Node key** is derived from the *extended node key* with 
`/chain'/0'/node_index'` derivation, where `chain'` is a hardened index 
specifying blockchain operated by the node, and `node_index'` is a hardened
incremental index for the nodes managed by a seed owner.

**Channel basepoint** is derived from the *extended node key* using
`/chain'/1'/type'/channel'` derivation, where `channel'` is a hardened index 
constructed from the 31 most significant bits of the channel id and
`type'` is a hardened index set to `0'` for BOLT-defined lightning channels
and `1'` for channels under Bifrost standards.

**Funding wallet** used for keeping funds by a lightning node for 
constructing funding transactions is derived with 
`/chain'/2'/type'/case/index` path, where `type'` has the same meaning as in
*channel basepoint* derivation; `case` is an eqivalent of `change` field
with RGB extensions (see ___) and `index` is a sequential index number.

Derivation paths for the base points are the following:

Basepoint name | Derivation suffix from *channel basebpoint*
-------------- | -------------------------------------------
Payment        | `/1'`
Delayed        | `/2'`
Revocation     | `/3'`
Per-commitment | `/4'`
HTLC           | `/5'`
PTLC           | `/6'`


## Compatibility

TBD: investigate derivation paths used by specific lightning nodes

### c-Lightning

### LND

### Eclair

### LNP Node

LNP Node is fully compatible with the standard

### Electrum


## Rationale


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

