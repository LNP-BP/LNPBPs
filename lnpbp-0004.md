```
LNPBP: 0004
Layer: Transaction (1)
Field: Cryptographic primitives
Title: Multi-message commitment scheme with provable zero-knowledge properties
Author: Dr Maxim Orlovsky <orlovsky@pandoracore.com>
Comments-URI: https://github.com/LNP-BP/lnpbps/issues/8
Status: Draft
Type: Standards Track
Created: 2019-28-10
License: CC0-1.0
```

## Abstract

The standard defines a way to commit to a multiple independent messages with a single digest such that the fact of
each particular commitment and a protocol under which the commitment is made may be proven without exposing the 
information about the other messages and used protocols.


## Background and Motivation

LNPBP-3 defines a standard for embedding cryptographic commitment into bitcoin transaction in a deterministic provable 
way [lnpbp3]. The standard is based on LNPBP-1 public key tweaking procedure [lnpbp1], which prevents multiple 
commitments inside a tweak. However it is possible that some protocol may require to commit to a number of messages 
within a single transaction and public key with the requirement that some dedicated information from this messages (like 
the message type) should be unique across the whole message set. For instance, this is required for state updates, where
the updates are separated into different blocks (messages) and kept private, such that a single party will know 
information about a single update and should not be exposed any information about the rest. However, in such a case,
there should be a proof that other state updates does not affect the state of the analyzed update, excluding state 
collisions. In such a setup, each state may be assigned a unique integer identifier (like cryptographic digest) and
a special form of zero-knowledge proof should be utilized to proof the fact that all of the states are different
without exposing the actual state ids. While this is impossible at the level of LNPBP-3 & LNPBP-1 standards, the current
proposal defines a procedure for structuring multiple independent messages in a privacy-preserving (zero-knowledge)
way, allowing that some properties of the committed messages may be proven in a zero-knowledge way, i.e. without 
revealing any information about the the source messages or the properties themselves.


## Design

Let's represent a set of messages (like state update transactions) as a set of binary strings m_1 .. m_n. Let's define
p_1 .. p_n to be a property in form of a binary strings, that must uniquely identify those messages, such as no two 
messages may share the same value for the property, i.e. for any given p_i, p_j, i ≠ j -> p_i ≠ p_j.

In order to construct the proof, first we convert the properties into a 256-bit integers, for instance with 
cryptographic digest, like SHA256 hash function, such as d_i = SHA256(p_i). For a given digests d_1 .. d_n we create 
Pedersen commitments C_1 .. C_n with blinding factors b_1 .. b_n. Then, for each pair of the commitments, C_i and C_j, 
we do provide a co-factor C_i,j such as C_i - C_j = C_i,j. Next, we disclose b_i,j to a verifier, so it may check that 
C_i,j ≠ b_i,j * G, implying d_i - d_j = d_i,j ≠ 0 and proving d_i and d_j to be distinct numbers.

We also have to prove that blinding factors b_i,j are actual blinding factors used in construction of C_i,j, and not
just some random numbers. In order to do that we do another round of Pedersen commitments to the x-coordinates of 
Dx_i,j = (d_i,j * G).x, Bx_i,j = (b_i,j * G).x and Cx_i,j = (C_i,j).x points, so the verifier is able to make sure that 
the sum of the P_Bx_ij + P_Dx_ij commitments is equal to P_Cx_ij.

the commitments to d_i,j and b_i,j is equal to the C_i,j commitment.


## Specification

### Commitment procedure

Each message m_i and it's unique property p_i (like state indentifier, protocol identifier etc) are serialized as a 
binary strings according to some consensus serialization rules. Then, a double SHA256 hash is computed for each of the
message and property string, producing 256-bit values c_i and d_i correspondingly. For a given digests d_1 .. d_n 
Pedersen commitments C_1 .. C_n are created with random non-zero blinding factors b_1 .. b_n. The data are serialized
as a binary string in form of

`s = <C_1> <c_1> <C_2> <c_2> .. <C_n> <c_n>`

where each `<x>` block represents a 256-bit (8-byte) number with highest significant bytes first. The string is double
hashed with SHA256 tagged-hash function from [bip-schnorr] using tag 'LNPBP-4' and the resulting digest represents the 
actual commitment message that can be utilized with other protocols, like LNPBP-1.


### Partial-reveal and proof procedure

To proof the commitment for the message number `k` and the uniqueness of it's property `p_k` the committing party must 
provide verifying party with:
- the binary string `s`
- source message `m_k`
- value for the `p_k` property
- set of `n-1` blinding factors b_i,k for `i` in 1 .. n range with the exception of i=k case
- Pedersen commitments P_Bx_ik, P_Dx_ik and P_Cx_ik to b_i,k * G, d_i,k * G and C_i,k x-coordinate points

The verifying party must make sure that:
1. Tagged double SHA-256 hash of the string `s` corresponds to the actual commitment
2. The source message `m_k` double SHA-256 hash corresponds to the `c_k` value from the `s`
3. None of the differences between C_k and each other C_i commitment are equal to b_i,k * G values
4. Check that the sum of P_Bx_ik + P_Dx_ik = P_Cx_ik

This proves that the message `m_k` was commited under the procedure and no other message with the same property `p_k`
was a part of this multi-message commitment.


## Compatibility

TBD


## Rationale

TBD


## Reference implementation

TBD


## Acknowledgements

## References

lnpbp1. Maxim Orlovsky. Key tweaking: collision-resistant elliptic curve-based commitments (LNPBP-1 Standard). 
   <https://github.com/LNP-BP/lnpbps/blob/master/lnpbp-0001.md>
lnpbp3. Maxim Orlovsky. Deterministic definition of transaction output containing cryptographic commitment
    (LNPBP-3 Standard). <https://github.com/LNP-BP/lnpbps/blob/master/lnpbp-0003.md>
bip-schnorr. Pieter Wuille. Schnorr Signatures for secp256k1.
    <https://github.com/sipa/bips/blob/bip-schnorr/bip-schnorr.mediawiki>


## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.


## Test vectors

TBD
