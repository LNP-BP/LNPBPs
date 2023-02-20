```
LNPBP: 0006
Vertical: Bitcoin protocol
Title: Tapret: taproot-based OP_RETURN deterministic bitcoin commitments
Author: Dr Maxim Orlovsky <orlovsky@lnp-bp.org>,
        Peter Todd
Comments-URI: https://github.com/LNP-BP/discussions
Status: Draft
Type: Standards Track
Created: 2022-04-01
Finalized: not yet
License: CC0-1.0
Related standards: LNPBP-4, BIP-341
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

The proposal provides standard mechanism for creating, structuring & verifying
deterministic bitcoin commitments, which puts the commitment in a leaf tapscript
with `OP_RETURN` opcode in the taproot script tree. 

## Background

TBD

## Motivation

TBD

## Design

Tapret commitment is structured as an `OP_RETURN`-based script, containing the
commitment to the LNPBP-4 message, constructed under multiple protocols. The
commitment script (**tapret leaf script**) always consists of 
[64 bytes][why-64-bytes].

The leaf with the tapret leaf script is always put into the same depth of the
taproot script tree, and this [depth is 1][why-depth-1] (in 0-based indexing of 
depth levels, where depth 0 corresponds to the merkle tree root). The commitment
is always put into the rightmost node of the tree by using consensus ordering
of the nodes, as it is defined in BIP-341 merkle path construction 
(lexicographic ordering). This precise definition of the possible place of the
commitment allows to prove the uniqueness of the commitment, i.e. the absence of 
any alternative tapret commitment in the same tree.

If at depth 7 in the rightmost position of the tree another taproot node is 
present (either leaf script or branch), an additional branch node is created 
with both tapret leaf script and the parent of the rightmost depth-7 node 
becoming its children. This new branch node is added instead of the original 
parent at depth 6, efficiently shifting the previous subtree from that position
one level down.

If the original taproot script tree does not have depth 7 nodes on its
right-side (in terms of consensus lexicographic ordering of each branch child
hashes), a subtree (**tapret subtree**) consisting of repeated tapret script 
leaves is created and inserted at the depth of the last node of the tree on the 
right-side merkle path. The height of the subtree is selected to put the taproot
script leaves at depth 7.

Since addition of the new node will change the merkle hashes of all its parents, 
and the merkle hash value is used in the lexicographic ordering of the tree, 
this operation with `1 - 1/(2 ^ 7) = 99.22%` probability will make just inserted
tapret subtree merkle root to be non-rightmost node of the modified tree. Since
the deterministic nature of the tapret commitments requires ability to prove the
absence of any alternative commitment in the same tree, two components are used
to keep the determinism of the commitment position and ability to prove its
uniqueness.

First, a special one-byte variable (nonce) is added to the taproot leaf script,
which allows "mining" the hash value in a way that the added subtree will 
appear at the right-side of the tree. At depth 7 there might be only 2^7=128
possible tree position, and 256 iterations (per one nonce value) should be 
enough to solve the issue.

Second, if it was not possible to solve the issue with the nonce, a special
**uniqueness proof** is produced, which ensures that none of the nodes at 
depth 7 on the right side of the tree does not contain alternative commitment.
This proof includes leaf script (for leaf nodes) or two child node hashes
(for branch nodes) for each of the nodes right to the tapret leaf script.

The tapret commitment is put into a transaction output `scriptPubkey` as a 
modified BIP-341 output key value, which is produced from the same internal key
and new merkle root of the taproot script tree containing the embedded tapret
commitment. The number of the transaction output with the commitment, if
multiple taproot outputs are present in the same transaction, must be 
deterministically defined by an upper-level protocol using the present tapret
commitment scheme.

Internal key value, merkle proof and uniqueness proof are combined to construct
**tapret proof**, passed to the validators as off-chain data for 
client-side validation.


## Specification

### Tapret proof

Tapret proof must consist of the following data, serialized exactly in the 
given order:
1. 32 bytes: internal key value, representing x-only public key serialized 
   according to BIP-341;
2. 1 byte: **nonce value** used in constructing tapret script;
3. 1 byte: number `n` of merkle proof elements (see below), must be in range
   `0..=7`, otherwise the proof must be considered invalid;
4. `n*32` bytes: sequence of **merkle proof** elements, ordered by their depth 
   in the tree. Each of the elements consists of 32 bytes representing hash of 
   the taproot script tree hashing partner;
5. 1 byte: number `u` of uniqueness proof elements (see below), must be less or
   equal to `2^(n+1) - 2`, otherwise the proof must be considered invalid;
6. sequence of `m` **uniquness proofs**, each consisting of: 
   6.1. 1 byte: proof type, either `0x00`, `0x01` or `0x02`. The proof type
        defines length of the proof data and their semantic meaning:
   6.1. the proof type `0x00` (called `empty`) means that node is absent at the 
        depth 7. The proof type is followed by one byte indicating the depth of 
        the last script leaf present in the tree, followed by the structure 
        described in pt. 6.3. below;
   6.2. for the proof type `0x01` (called `branch`) two 32-byte hash values
        are given; these hash values represents hashes of taproot script
        tree nodes;
   6.3. for the proof type `0x02` (called `leaf`) a following structure is used:
        6.3.1. 1 byte containing script leaf version,
        6.3.2. 2 bytes containing script length `l` in little endian encoding,
        6.3.3. `l` bytes of raw leaf script data.


### Tapret verification

First, a **tapret script leaf** is constructed. The version of the script must
be `0xC0` (tapscript) and the script must be equal to the byte sequence (in hex)
`FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF01<nonce>6A20` 
followed by a 32-byte serialization of LNPBP-4 multiprotocol commitment.
The provided byte sequence represent tapscript consisting of 28 
`OP_INVALIDOPCODE`, followed by `OP_PUSHBYTES_1`, nonce byte (taken from tapret
proof), `OP_RETURN` and `OP_PUSHBYTES_32` op-codes[^why-64-bytes].

Then, **tapret subtree** is constructed from the *tapret script leaves*. The 
tree should be a valid taproot script tree under BIP-341 standard, consisting of
`8-n` depth levels, where at depth = 7 (for zero-based depth indexing, 
representing merkle root level) the subtree must contain `2^(7-n)` *tapret 
script leaves*. For the edge case of `n=7` the subtree must consist of a single 
*tapret script leaf* which `TapLeaf`-tagged hash will be the hash of the subtree
merkle root.

The merkle root hash of the *tapret subtree* is used together with the provided
node hash partner values from the perkle proof data of the tapret proof to 
receive the final merkle tree root value. Combined with the value of the 
internal key from the tapret proof according to BIP-341 algorithm it must 
produce exactly the same output key value as present in the `scriptPubkey` of
the transaction output containing the commitment; otherwise validation procedure
fails.

Finally, the validity of the uniqueness proof must be checked. The uniqueness
proof data are used to reconstruct each and all the right-side merkle tree 
hashing partners, which must match the exact number and values of the hasing
partners from the merkle proof, which values were grater than the value of the
parent of the tapret leaf script at that depth. If this procedure fails, the
validation procedure must fail.


### Tapret construction


## Compatibility

TBD

## Reference implementation

TBD

## Rationale

### Tapret proof size estimation

Due to the use of nonce "mining" mechanism, in the most of cases the proof size
should not exceed 290 bytes. Since the most of the taproot script trees will not
have a depth more than 1 or 2, the actual size in these cases will be 66 or 98 
bytes.

### Why 64 byte tapscript

This allows distinguishing of tapret leaf script from a data used in production
of taproot branch hash, such that the proof of the absence of an alternative
tapret commitment can be validated by simple comparison of the first of the
child node hashes to the tapret leaf script prefix.

### Why script depth 1

This helps to keep client-side-validated proof size smaller.


## Acknowledgements

TBD

## References

TBD

## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.

## Test vectors

TBD

[why-64-bytes]: #why-64-byte-tapscript
[why-depth-1]: #why-script-depth-1
