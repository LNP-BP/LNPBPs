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
Payment        | `/1'`             | `m/9735'/chain'/1'/ln_ver'/channel'/1'`
Delayed        | `/2'`             | `m/9735'/chain'/1'/ln_ver'/channel'/2'`
Revocation     | `/3'`             | `m/9735'/chain'/1'/ln_ver'/channel'/3'`
Per-commitment | `/4'`             | `m/9735'/chain'/1'/ln_ver'/channel'/4'`
HTLC           | `/5'`             | `m/9735'/chain'/1'/ln_ver'/channel'/5'`
PTLC           | `/6'`             | `m/9735'/chain'/1'/ln_ver'/channel'/6'`
Shutdown       | `/7'`             | `m/9735'/chain'/1'/ln_ver'/channel'/7'`

**Funding wallet** used for keeping funds by a lightning node for 
constructing funding transactions is derived with 
`/chain'/2'/case/index` path, where `case` is an eqivalent of `change` field
with RGB extensions (see ___) and `index` is a sequential index number.

Example of extended keys used by lightning node #0 on bitcoin network
for a BOLT-defined channel with temporary channel id 
`2938ce5c0cae4b2af072065dc7dcea68de5e3de0c936742e27e045c8650c6a79`:

Extended keys            | Derivation
------------------------ | -----------
Lightning key            | `m/9735'/`
Node key                 | `m/9735'/0'/0'/0'`
Channel basepoint        | `m/9735'/0'/1'/0'/691588700'` (index equals to `0x2938ce5c`)
Funding wallet           | `m/9735'/0'/2'/0/*`
Funding wallet change    | `m/9735'/0'/2'/1/*`
Funding wallet RGB20     | `m/9735'/0'/2'/20/*`


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

