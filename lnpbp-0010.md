```
LNPBP: 0010
Vertical: Bitcoin protocol
Title: Bitcoin transaction output-based single-use-seals
Author: Dr Maxim Orlovsky <orlovsky@lnp-bp.org>,
        Peter Todd,
        Giacomo Zucco,
        Federico Tenga,
        Martino Salvetti
Comments-URI: https://github.com/LNP-BP/lnpbps/pulls/117
Status: Draft
Type: Standards Track
Created: 2021-11-15
Finalized: not yet
License: CC0-1.0
Related standards: LNPBP-1, 2, 3, 4, 7, 8, 9, 6, 92
```

- [Abstract](#abstract)
- [Background](#background)
- [Motivation](#motivation)
- [Design](#design)
- [Specification](#specification)
  - [Single-use-seal definition](#single-use-seal-definition)
  - [Single-use-seal concealment](#single-use-seal-concealment)
  - [Closing single-use-seal](#closing-single-use-seal)
  - [Proving single-use-seal close](#proving-single-use-seal-close)
- [Privacy considerations](#privacy-considerations)
  - [UTXO and TxO ownership](#utxo-and-txo-ownership)
  - [Chain analysis](#chain-analysis)
  - [Private data](#private-data)
- [Security considerations](#security-considerations)
  - [Hardware wallets](#hardware-wallets)
- [Compatibility](#compatibility)
  - [Version 1 transactions](#version-1-transactions)
  - [Replace-by-fee](#replace-by-fee)
  - [CheckSequenceVerify](#checksequenceverify)
  - [Bare script outputs](#bare-script-outputs)
  - [Segregated witness v0](#segregated-witness-v0)
  - [Taproot](#taproot)
  - [Future segregated witness variants](#future-segregated-witness-variants)
  - [Bitcoin sidechains](#bitcoin-sidechains)
  - [Pay-to-address](#pay-to-address)
  - [Hardware wallets](#hardware-wallets-1)
  - [Software wallets](#software-wallets)
- [Rationale](#rationale)
  - [Why both S2C and P2C schemes are supported](#why-both-s2c-and-p2c-schemes-are-supported)
  - [Why multiple nSeq bits are used](#why-multiple-nseq-bits-are-used)
  - [Why signal commitment scheme with nSeq field](#why-signal-commitment-scheme-with-nseq-field)
  - [Why fee is used in determining commitment point](#why-fee-is-used-in-determining-commitment-point)
- [Reference implementation](#reference-implementation)
- [Acknowledgements](#acknowledgements)
- [References](#references)
- [Copyright](#copyright)
- [Test vectors](#test-vectors)


## Abstract

The proposal standardises single-use-seal-based cryptographic commitment scheme
utilizing bitcoin blockchain and bitcoin transaction graph as a 
proof-of-publication medium, using bitcoin transaction outputs as seal
definitions and spending bitcoin transactions as proofs of seal close and
containers for a commitment to a message over which the seal is closed.
The verification procedure requires storing part of the single-use-seal
witness data on the client side (*extra-transaction proof*).


## Background

Single-use-seal is a cryptographic commitment primitive originally proposed
by Peter Todd in 2016 and standardized as LNPBP-8 standard. However, Here we 
define a specific commitment scheme based on that standard using bitcoin
transactions.


## Motivation


## Design

Bitcoin transaction outputs are used as a seal definitions (*TxO seal*), 
consisting of transaction id and 32-bit output number. Other bitcoin 
transaction (*witness transaction*) spending previous output defined as a 
single-use-seal operates as a part of the witness of the seal being closed over
certain message and contains a commitment to that message, deterministically 
structured either in form of sign-to-contract (S2C) or pay-to-contract (P2C) 
commitment using LNPBP-1, 2, 3, 92 commitment schemata. Which specific 
commitment scheme (S2C or P2C) is used determined by `nSeq` value of the
witness transaction input spending TxO seal. The message iself is structured as
an LNPBP-4 multi-message commitment, which allows embedding of multiple 
commitments under different protocols inside the same seal witness.


## Specification

Single-use-seal related terms in this specifications are defined according to
LNPBP-8 standard.

### Single-use-seal definition

### Single-use-seal concealment

### Closing single-use-seal

### Proving single-use-seal close


## Privacy considerations

### UTXO and TxO ownership

### Chain analysis

### Private data


## Security considerations

### Hardware wallets


## Compatibility

### Version 1 transactions

### Replace-by-fee

### CheckSequenceVerify

### Bare script outputs

### Segregated witness v0

### Taproot

### Future segregated witness variants

### Bitcoin sidechains

### Pay-to-address

### Hardware wallets

### Software wallets


## Rationale

### Why both S2C and P2C schemes are supported

### Why multiple nSeq bits are used

### Why signal commitment scheme with nSeq field

### Why fee is used in determining commitment point




## Reference implementation

The reference implementation is version `1.0` of [`bp-seals` crate](bp-seals), 
which is a part of the [BP Core Library](bp-core) developed and maintained 
by LNP/BP Strandards Association.

If the implementation differs from the spec, the spec has a higher priority.


## Acknowledgements


## References


## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.


## Test vectors


[bp-seals]: https://crates.io/crates/bp-seals
[bp-core]: https://github.com/LNP-BP/bp-core
