```
LNPBP: 0006
Vertical: Bitcoin protocol
Title: Deterministic bitcoin commitments
Author: Dr Maxim Orlovsky <orlovsky@lnp-bp.org>
Comments-URI: https://github.com/LNP-BP/lnpbps/pulls/116
Status: Rejected
Type: Standards Track
Created: 2021-11-15
Finalized: not yet
License: CC0-1.0
Related standards: LNPBP-1, 2, 4, 7, 9, 92
```

## Abstract

The proposal provides standard mechanism for creating, structuring & verifying
deterministic bitcoin commitments ("anchoring") to multiple extra-transaction
data (client-side-validated etc).

## Background

## Motivation

## Design

Miltiple protocols may commit to external (non-transaction and non-blockchain) 
data by constructing a special data structure called *anchor* and committing
to its content inside transaction input or output.


## Specification

### Commitment procedure

A party needing to commit to multiple data coming from distinct protocols
MUST follow the algorithm:
1. Construct LNPBP-4 multi-message commitment;
2. Deterministically define transaction input or output which will contain
   the final commitment. This algorithm is not a part of the current
   specification and must be defined at upper protocol level (see for instance
   LNPBP-10).
3. Run commitment embedding procedure:
   - If commitment will be placed into transaction output, construct
     *extra-transaction proof* using procedure from LNPBP-2; fail algorithm
     if the LNPBP-2 requirements can't be fulfilled.
   - If commitment will be placed into transaction input, construct
     *extra-transaction proof* using procedure from LNPBP-92; fail algorithm
     if the LNPBP-92 requirements can't be fulfilled.
4. Construct *anchor* data structure (see below), encode it and persist
   the data as long as it will be required to provide proofs of the commitment.

If algorithm failed the party creating commitment MAY change the structure
of the transaction and try to repeat the commitment procedure.

### Verification procedure

The verification process runs by repeating steps of the commitment protocol
using the information provided during the *reveal phase* and verifying the
result is equal to the provided proofs. If the commitment procedure fails
at any of the steps, the verification procedure MUST also fail.

### Anchor structure and serialization

Anchor consists of the following fields, serialized according to the strict
serialization rules (see LNPBP-7):
1. `txid`: 32-byte transaction id
2. `commitment`: a commutment block structure from LNPBP-4
3. `proof`: a deterministic commitment proof which has the same structure for 
   both LNPBP-2 and LNPBP-92 commitment:
   1. `original_pubkey`: original 33-byte value of the pubkey before tweak;
       for pay-to-signature (LNPBP-92) tweaks this is `R` value generated
       from a nonce. BIP-340 keys MUST be encoded as 33 byte value having
       first byte `0x02`.
   2. `script_info`: an enum, encoded with a first byte representing enum 
       variant
       - `0x00 = SinglePubkey`: always used by LNPBP-92 and by LNPBP-2
          commitments based on P2PK, P2PKH, P2WPKH, legacy SegWit v0 
          P2WPKH-in-P2SH, P2TR with key spending only (self-tweaked), 
          OP_RETURN and other bare script outputs.
       - `0x01 = LockScript`: a Bitcoin script source which knowledge
         is required to satisfy spending for pre-SegWit P2SH, native SegWit v0 
         P2WSH. The enum variant byte MUST be followed by a strict-encoded
         representation of binary script data. This script MUST match
         `redeemScript` for P2SH and `witnessScript` for P2WSH.
       - `0x02 = WrappedWitnessScript`:  Bitcoin script source which knowledge
         is required to satisfy spending for legacy (P2SH-wrapped) SegWit v0,
         P2WSH-in-P2SH outputs (and, potentially, fututre SegWit versions).
         The enum variant byte MUST be followed by a strict-encoded
         representation of binary script data. This script MUST match
         `witnessScript` for P2WSH; for P2SH wrapped version it also must
         be a `witnessScript` and not `redeemScript`.
       - `0x03 = TaprootScript`: a Merkle root of the script spending path
         of the taproot output or a value of a tweak applied to the internal
         taproot key if and only if this tweak is not a self-tweak. Must be
         followed by exactly 32 bytes (without length prefix) of the tweak
         data in little endinan order.

The list of `script_info` variants is non-exhaustive. If a future version of 
script info variant byte is met the deserialization of anchor data MUST fail 
with explicit error and the anchor MUST not be verified.

If the anchor data does not match the provided script pubkey (for instance,
being encoded as P2WSH-in-P2SH instead of P2WSH) the verifying party MUST 
fail verification even if other option is satisfied with other (non-specified)
encoding.

### Concealment of the anchor data

Anchor data contain extra information which is not required for the DBC 
validation and represents additional proofs for non-DBC validation procedures.
This information is contained in `commitment` field of the *anchor* data
structure containing LNPBP-4 multi-message commitment entrypy value, used
for proving the absence of some specific protocol within the commitment tree.
This information MAY be discarded for privacy reasons when the *anchor* data
are passed to a third-party with a special *conceal* procedure as described
in LNPBP-4 and LNPBP-9 standards.


## Compatibility

## Rationale

### Why not to use `0x03` variant for keeping key spend path-only Taproot info

Using `0x03` variant (`TaprootScript`) for Taproot outputs having no script
path spending conditions (i.e. self-tweaked keys) instead of `0x00` 
(`SinglePubkey`) will result in storing additional 32 bytes on the client side
per each output of this type, while providing no privacy gains: a user
hacing access to the anchor data can always make sure that the value of the
script tweak is equal to the self-tweak and still acquire knowledge that
the output does not contain a script path spending condition.

### Why not to use single `0x01` variant for all types of scripts

P2WSH-in-P2SH require its own script info variant (`0x02`, 
`WrappedWitnessScript`) since this allows to simplify verification algorithm
by avoiding trying multiple script encodings (native P2SH and legacy 
P2WSH-in-P2SH). This does not reduce forward compatibility: we may have 252
more variants for script info value, which is exceeds a possible number of 
future bitcoin script pubkey encoding (15 possible future witness values).

## Reference implementation

The reference implementation is version `1.0` of [`bp-dbc` crate](bp-dbc), 
which is a part of the [BP Core Library](bp-core) developed and maintained 
by LNP/BP Strandards Association.

If the implementation differs from the spec, the spec has a higher priority.

## Acknowledgements

## References

## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.

## Test vectors

[bp-dbc]: https://crates.io/crates/bp-dbc
[bp-core]: https://github.com/LNP-BP/bp-core
