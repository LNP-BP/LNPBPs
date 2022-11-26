```
LNPBP: 0005
Vertical: Bitcoin protocol
Title: Universal short Bitcoin identifiers for blocks, transactions and
       transaction inputs & outputs
Author: Dr Christian Decker <decker.christian@gmail.com>,
        Dr Maxim Orlovsky <orlovsky@protonmail.ch>
Comments-URI: https://github.com/LNP-BP/lnpbps/issues/<____>
Status: Proposal
Type: Standards Track
Created: 2020-02-17
License: CC0-1.0
```

- [Abstract](#abstract)
- [Background](#background)
- [Motivation](#motivation)
- [Specification](#specification)
  - [Bitwise structure of the identifier](#bitwise-structure-of-the-identifier)
  - [Distinguishing different types of identifier](#distinguishing-different-types-of-identifier)
- [Compatibility](#compatibility)
- [Rationale](#rationale)
  - [Estimating ranges for bit-based values](#estimating-ranges-for-bit-based-values)
- [Reference implementation](#reference-implementation)
- [Acknowledgements](#acknowledgements)
- [References](#references)
- [Copyright](#copyright)
- [Test vectors](#test-vectors)


## Abstract

The work introduces short 64-bit identifiers able to uniquely distinguish
bitcoin blocks, transactions, transaction inputs and outputs for both on-chain
transactions and off-chain transactions, representing sufficient collision
resistance for the common tasks of  blockchain and transaction data indexing,
search and compact archive storage. This format may be useful for different
Layer 2 and 3 applications, as well as user-based clients and indexing services.


## Background

Bitcoin transactions are normally identified by 256-bit integers, representing
double SHA256 hashes of the conservative transaction data serialized according
to the original Satoshi Bitcoin client code [1] and SegWit BIP-141 specification
[2]. These identifiers are unique: two distinct transactions has probability of
~2^256 that their identifiers would collide. While such security is undoubtedly
required to maintain global identifiers for all possible  transactions, for the
transactions already included in Bitcoin blockchain it may be unnecessary for
multiple cases. The main drawback of using 256-bit integers is inconveniences
for normal computing tasks: modern CPUs and databases can easily handle up to
64-bit integers, but not 256-bit. This introduces unnecessary load for indexing
and search tasks; increases storage requirements and network traffic.

Lightning network protocol (LNP) has introduced the use of short channel
identifiers [3],  a 64-bit integers composed of the channel funding transaction
data:

* block height (occupying the most significant 3 bytes)
* transaction index within the block (the next 3 bytes)
* output index funding the channel (least significant 2 bytes)

The introduction of this data structure was reasonable, since Lightning network
operates only channels funded with transactions secure from deep re-orgs, thus
introduction of blockchain-based identifiers was safe.

Here we propose more generic scheme, that follows the design ideas from LNP
extending the concept of short identifiers beyond addressing transaction outputs
only.


## Motivation

Normally, references to blockchain blocks, transactions, their inputs and
outputs occupy 32 - or up to 37 bytes (in case of inputs/outputs). For the
existing set of transactions residing in blockchain just listing all existing
data will  require gigabytes of storage space. Many user applications and
services, However, require cross-referencing and indexing - like `JOIN` SQL
queries connecting different tables (like transactions, outputs and inputs) with
primary/foreign keys, where the keys are 32-byte values. This is both extremely
inefficient in terms of performance and index storage space. While it may be a
normal practice to use incremental 32-bit indexing, the full identifiers still
should be kept, and two different databases created by the same software after
some re-orgs will not match by their indexes.

Thus, we are proposing a format for 64-bit identifiers, that are both collision-
and reorg-resistant (with ~1/256 probability) and can be used for an efficient
indexing, primary/foreign key values and referencing.


## Specification

### Bitwise structure of the identifier

The identifier is a 64-bit number, where  value of the most significant bit
defines format for the rest of the bits. This first bit represents a flag
distinguishing on-chain and off-chain transactions and items. If the flag is
set to 0, the bits MUST have the following meaning:

```
_ _ _ _  _ _ _ _   _ _ _ _  _ _ _ _   _ _ _ _  _ _ _ _   _ _ _ _  _ _ _ _   _ _ _ _  _ _ _ _   _ _ _ _  _ _ _ _   _ _ _ _  _ _ _ _   _ _ _ _  _ _ _ _
^ ^                                                  ^   ^              ^   ^                                 ^   ^ ^                               ^
| |                                                  |   |              |   |                                 |   | |                               |
| +--------------------------------------------------+   +--------------+   +---------------------------------+   | +-------------------------------+
|                 Block height, 23 bits                 BlockHash checksum   Transaction position in the block    |     TxIn/TxOut index, 15 bits
Off-chain flag, set to 0                                      8 bits                     16 bits                  TxIn/TxOut flag
```

| Bits* | No bits | Possible no of values | Meaning |
|------:|--------:|----------------------:|---------|
|     0 |       1 | 2         | Flag indicating the structure for the rest of bits; MUST be set to 0 for the provided case |
|  1-23 |      23 | 8'388'608 | Block height; sufficient to cover >160 years of blockchain history |
| 24-31 |       8 | 256       | Checksum for block hash value: binary XOR of it's 8 bytes |
| 32-47 |      16 | 65'536    | Transaction position in the block; with 1 MB block size limit sufficient to cover all transactions even if their average size is 15 only |
|    48 |       1 | 2         | Flag indicating whether the next 15 bits represents transaction input (0) or output (1) index |
| 49-63 |      15 | 32'768    | Transaction input or output index (starting from 1) |

* In "most significant bit goes first" order


If the most significant bit of the short identifier is set to 1, the bits MUST
have the following meaning:

```
_ _ _ _  _ _ _ _   _ _ _ _  _ _ _ _   _ _ _ _  _ _ _ _   _ _ _ _  _ _ _ _   _ _ _ _  _ _ _ _   _ _ _ _  _ _ _ _   _ _ _ _  _ _ _ _   _ _ _ _  _ _ _ _
^ ^                                                                                                           ^   ^ ^                               ^
| |                                                                                                           |   | |                               |
| +-----------------------------------------------------------------------------------------------------------+   | +-------------------------------+
|                                          Highest 47 bits from transaction id                                    |     TxIn/TxOut index, 15 bits
Off-chain flag, set to 1                                                                                          TxIn/TxOut flag
```

|Bits* | No bits | Possible no of values | Meaning  |
|------:|--------:|----------------------:|---------|
|    0 |       1 | 2         | Flag indicating the structure for the rest of bits; MUST be set to 1 for the provided case |
| 1-47 |      47 | >140 trillions (1.4*10^14) | Highest 47 bits from transaction id |
|   48 |       1 | 2         | Flag indicating whether the next 15 bits represents transaction input (0) or output (1) index |
|49-63 |      15 | 32'768    | Transaction input or output index (starting from 1) |

* Using big endian (network) byte order

### Distinguishing different types of identifier

The identifier may be used to denote 7 different types of Bitcoin blockchain-
related entities:

Entity             | Kind      | Distinguishing factor
-------------------|-----------|-----------------------
Block              | On-chain  | First bit set to 0, bits 32-63 set to 0
Transaction        | On-chain  | First bit set to 0, bits 48-63 set to 0
Transaction input  | On-chain  | First bit set to 0, bit 48 set to 0, at least one of the bits in range of 49-63 is a non-zero
Transaction output | On-chain  | First bit set to 0, bit 48 set to 1, at least one of the bits in range of 49-63 is a non-zero
Transaction        | Off-chain | First bit set to 1, bits 48-63 set to 0
Transaction input  | Off-chain | First bit set to 1, bit 48 set to 0, at least one of the bits in range of 49-63 is a non-zero
Transaction output | Off-chain | First bit set to 1, bit 48 set to 1, at least one of the bits in range of 49-63 is a non-zero


It should be noted, that coinbase transactions has the same identifier as the
block itself (since transaction indexes within the block are 0-based, unlike
transaction input and output indexes, which are 1-based): in this regard
coinbase transaction can be seen as a natural part of the block data.
Nevertheless, identifiers for coinbase transaction outputs and inputs are
distinguished from the unique short block identifier.


## Compatibility

The proposed format is incompatible with previously existing block, transaction,
transaction input and output identifiers, including `short_channel_id` from
Lightning network protocols.

The format is highly compatible with the requirements of modern databases,
including both SQL and No-SQL databases, as well as generic CPU capabilities and
programming languages, since all of them are able to deal with 64-bit numbers in
highly efficient manner.


## Rationale

### Estimating ranges for bit-based values

The minimum size of transaction input is:

* 32 bytes reference to txid spent by this input
* 4 bytes for index of transaction output spent by this input,
* 1 byte for variable-length int denoting zero-size `sigScript`,
* 4 bytes for `nSeq`,

41 byte in total.

With 1 MB block size limit and the only one transaction fitting into the block,
it may have at most 2^20 / 41 <= 26'000 transaction inputs; which is  lower than
the maximum value of 15-bit number referencing transaction input index (2^15 =
32'768).


The minimum size of transaction output is:

* 8 bytes for amount (in satoshis)
* 1 byte for variable-length int denoting zero-size `scriptPubkey`,

for nonce/anyone can spent output, or at least

* 8 bytes for amount (in satoshis)
* 1 byte for `scriptPubkey` length fields
* 33 bytes for the shortest possible spendable version of `scriptPubkey`
  controlled by public key(s): 1 byte SegWit push of `0x01` (for Taproot)
  followed by 32-bytes of witness program [2], representing a serialized Taproot
  public key according to [5].

I.e. 42 bytes in total, meaning that the same equation valid for transaction
inputs above stands for transaction outputs: no meaningful bitcoin transaction
can't hold more than 2^15 outputs with the current validation rules.


## Reference implementation

<https://github.com/LNP-BP/rust-lnpbp/pull/14>


## Acknowledgements



## References

1. <https://github.com/bitcoin/bitcoin>
2. Eric Lombrozo, Johnson Lau, Pieter Wuille. BIP-141: Segregated Witness
   (Consensus layer).
   <https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki#Transaction_ID>
3. BOLT #7: P2P Node and Channel Discovery.
   <https://github.com/lightningnetwork/lightning-rfc/blob/master/07-routing-gossip.md#definition-of-short_channel_id>
4. Pieter Wuille, Jonas Nick, Anthony Towns. BIP-341 Taproot: SegWit version 1
   output spending rules.
   <https://github.com/bitcoin/bips/blob/master/bip-0341.mediawiki>
5. Pieter Wuille, Jonas Nick, Tim Ruffing. BIP-340: Schnorr Signatures for
   secp256k1.
   <https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki>


## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.


## Test vectors

TBD
