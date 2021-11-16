```
LNPBP: 0092
Vertical: Bitcoin protocol
Title: Deterministic embedding of cryptographic commitments into bitcoin  
       transaction input
Author: Dr Maxim Orlovsky <orlovsky@lnp-bp.org>
Comments-URI: https://github.com/LNP-BP/lnpbps/pulls/<____>
Status: Draft
Type: Standards Track
Created: 2021-11-15
Finalized: not yet
License: CC0-1.0
```

- [Abstract](#abstract)
- [Background](#background)
- [Motivation](#motivation)
- [Design](#design)
- [Specification](#specification)
- [Compatibility](#compatibility)
- [Rationale](#rationale)
- [Reference implementation](#reference-implementation)
- [Acknowledgements](#acknowledgements)
- [References](#references)
- [Copyright](#copyright)
- [Test vectors](#test-vectors)


## Abstract

The proposal standardizes sign-to-contract commitmnets in bitcoin transaction
inputs, proposing a robust and efficient scheme. It defines commitment and
verification protocol, scope of applicable conditions when the commitment
can be created and provides specific of client-side data structures used for
keeping and validating commitment proofs.


## Background


## Motivation


## Design

Sing-to-contract commitments are embedded into transaction outputs depending
on the type of the spent `scriptPubkey` and fields in the spending input.
For always-single-public key types (P2PK, P2PKH, P2WPKH, P2TR key path spending,
P2WPKH-in-P2SH) the only signature present in the `sigScript` or `witness`
will contain the commitment. For script-based types (bare scripts, native P2SH, 
P2WSH-in-P2SH, P2WSH and P2TR script path spendings), which may have from
zero to any number of signatures a more elaborated scheme is used, where
sign-to-contract commitment is embedded into any single signature, but the
verification of the commitment is made against the sum of all signature
nonces. Commitment procedure in this case also requires provable tweaking
of other signature nonces with their own tagged hashes. 

The commitment procedure results in creation of additional data structure,
*extra-transaction commitment proof*, which must be kept on the client-side
and is required for the commitment validation procedure.

This commitment scheme does not support script spendings when the tx input
contains no signatures, as well as P2TR script path spendings containing either
Annex or using non-BIP-342 (TapSCript) scripting. In this cases commiter shell
use other commitment schemes (like LNPBP-2) ensuring proper signalling like
it is done in LNPBP-10 scheme.


## Specification


## Compatibility


## Rationale

## Reference implementation

## Acknowledgements

## References

## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.

## Test vectors
