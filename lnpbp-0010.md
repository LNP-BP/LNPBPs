# LNPBP-39: TxO seals

```
LNPBP: 0010
Vertical: Bitcoin protocol
Title: Bitcoin transaction output-based single-use-seals
Authors: Dr Maxim Orlovsky <orlovsky@pandoracore.com>,
         Peter Todd <pete@petertodd.org>,
         Federico Tenga, 
         Giacomo Zucco
Comments-URI:
Status: Proposal
Type: Standards Track
Created: 2019-05-12
License: CC0-1.0
```

* [Abstract](#abstract)
* [Background](#background)
* [Motivation](#motivation)
* [Design](#design)
* [Specification](#specification)
  * [Terms and definitions](#terms-and-definitions)
  * [Seal definition](#seal-definition)
  * [Closing seal over a message](#closing-seal-over-a-message)
  * [Verification](#verification)
  * [Single-use-seal concealment](#single-use-seal-concealment)
  * [Strict encoding](#strict-encoding)
* [Privacy considerations](#privacy-considerations)
  * [UTXO and TxO ownership](#utxo-and-txo-ownership)
  * [Chain analysis](#chain-analysis)
  * [Private data](#private-data)
* [Compatibility](#compatibility)
  * [Version 1 transactions](#version-1-transactions)
  * [Pre-segwit outputs](#pre-segwit-outputs)
  * [Segregated witness v0](#segregated-witness-v0)
  * [Taproot](#taproot)
  * [Future segregated witness variants](#future-segregated-witness-variants)
  * [Bitcoin sidechains](#bitcoin-sidechains)
  * [Pay-to-address](#pay-to-address)
  * [Hardware wallets](#hardware-wallets)
  * [Software wallets](#software-wallets)
* [Rationale](#rationale)
  * [Representation of transaction output numbers](#representation-of-transaction-output-numbers)
* [Reference implementation](#reference-implementation)
* [Acknowledgements](#acknowledgements)
* [References](#references)
* [Copyright](#copyright)
* [Test vectors](#test-vectors)

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

Bitcoin transaction output-based single-use-seals (TxOSUS, short for Transaction Output Single Use Seals) is a particular application of [single-use-seals](lnpbp-0006.md) to bitcoin transaction graph, either as a part of any bitcoin blockchain (longest PoW chain, federated sidechain etc) or state channel (Lightning network channel and other types of state channels), i.e. in terms of single-use-seals, seal medium is represented by a bitcoin transaction graph.

The parties running protocol agree on some transaction output in any given bitcoin transaction graph as a place with special meaning ("seal"), and require that a future transaction spending this output ("witness transaction") contained a [deterministic bitcoin commitment](lnpbp-0008.md) to some a message. Any independent party having access to the transaction graph MUST:

1. not be able to detect the presence of the commitment in a given transaction graph (_hiding_ property of TxOSUS) even if the original message is known,
2. being provided information about specific protocol used for message commitments and access to deterministic bitcoin proof data (see [LNPBP-8](lnpbp-0008.md)) be able to verify the commitment such as it will be valid only and only for the message with which the original commitment was created (_verifiably_ property of TxOSUS).

The defined procedure is not the only way in which bitcoin blockchain or transaction graph can be used as a seal medium for single-use seals (for instance, a seal may be defined as a first time some specific public key appears in the blockchain); however these options are not part of the current standard.

## Specification

### Terms and definitions

**Seal** or **defined seal**: Bitcoin transaction outpoint: a combination of transaction identifier (consensus-defined double SHA256 hash of fully-signed valid bitcoin transaction) and transaction output number. Transaction output number MUST be represented as a 32-bit unsigned integer[1](lnpbp-0039.md#Representation-of-transaction-output-numbers).

**Witness transaction**: transaction spending an output specified by a given seal.

**Closed seal**: a seal for which a transaction spending output matching seal definition is known. This transaction MAY be part of longest block chain, or MAY not be a part of it (existing only within a Lightning network channel or as a part of any other off-chain transaction graph). It's up to particular implementation to decide whether such transaction (named **witness transaction**) should be considered valid or not; however this rules MUST require that the witness transaction MUST be a valid bitcoin transaction (i.e. it can be validated with libbitcoinconsensus or Bitcoin Core instance).

### Seal definition

Defined seal consists of 256-bit consensus-defined double SHA-256 hash of a fully-signed valid bitcoin transaction and 32-bit transaction output index.

Seal MUST be considered defined only and only all of transaction data are known, the transaction is fully signed and both transaction structure and signatures validated with bitcoin consensus rules using libbitcoinconsensus or Bitcoin Core.

NP: Parties participating the protocol MUST agree on an implementatio-specific entropy used in deterministic bitcoin commiemtnes as required by [LNPBP-8](lnpbp-0008.md) and [LNPBP-3](lnpbp-0003.md) protocols.

### Closing seal over a message

1. Fully sign the transaction containing output acting as a defined seal.
2. Construct a transaction spending the transaction output defined by the seal. The transaction may have any number
3. Create a commitment to the message and embed it into the unsigned witness transaction with the deterministic bitcoin commitment embed procedure as defined in [LNPBP-8](lnpbp-0008.md), store the proof.
4. Fully sign the witness transaction.

NB: Transactions and proof MUST be persisted for further verification procedure.

### Verification

A party verifying that the seal is indeed closed over a message MUST run the following procedure:

1. Make sure that the seal definition is fully known, otherwise fail the verification.
2. Verify transaction containing the defined seal:
  * the transaction MUST be a valid bitcoin transaction;
  * the transaction MUST contain all transaction data required for it's verification;
  * bitcoin-defined consensus transaction identifier MUST correspond to the transaction identifier provided as a part of seal definition;
  * transaction output number contained in the seal definition MUST be present in transaction (i.e. transaction MUST contain exactly that numbers or more of trsansaciton outputs);
  * the transaction MUST be fully-signed for all of its inputs;
  * the transaction MUST signatures pass validation with bitcoin consensus rules using libbitcoinconsensus or Bitcoin Core. If any of these conditions are not met, fail the verification.
3. Verify the witness transaction spending the transaction output defined as the seal subjected to the current verification procedure:
  * the transaction MUST be a valid bitcoin transaction;
  * the transaction MUST contain all transaction data required for it's verification;
  * the transaction MUST spend transaction output defined as the seal verified by the current procedure;
  * bitcoin-defined consensus transaction identifier MUST correspond to the transaction identifier provided as a part of seal definition;
  * transaction output number contained in the seal definition MUST be present in transaction (i.e. transaction MUST contain exactly that numbers or more of trsansaciton outputs);
  * the transaction MUST be fully-signed for all of its inputs;
  * the transaction MUST signatures pass validation with bitcoin consensus rules using libbitcoinconsensus or Bitcoin Core. If any of these conditions are not met, fail the verification.
4. Verify the deterministic bitcoin commitment to the given message with the provided deterministic bitcoin commitment proof according to the [LNPBP-8](lnpbp-0008.md) procedure; fail the verification if this procedure fails.

### Single-use-seal concealment

### Strict encoding

Any implementation of the standard MUST follow [strict encoding](../lnpbp-0007.md) rules, in particular

Defined seal MUST be serialized as fixed-length 36 byte-long structure:

* 32 bytes of transaction identifier encoded according to bitcoin transaction identifier consensus serialization rules in little-endian format
* 4 bytes of transaction output in little-endian format

Deterministic bitcoin commitment proofs MUST be serialized using strict encoding rules defined in [...].

There are no specific requirements to the message encoding or content; they MAY be defined as a part of other protocols using the current standard.

Bitcoin transactions and their parts, when needed, MUST be serialized according to bitcoin consensus serialization rules (which MAY differ from strict encoding requirements).

## Privacy considerations

### UTXO and TxO ownership

### Chain analysis

### Private data

## Compatibility

### Version 1 transactions

### Pre-segwit outputs

### Segregated witness v0

### Taproot

### Future segregated witness variants

### Bitcoin sidechains

### Pay-to-address

### Hardware wallets

### Software wallets

## Rationale

### Representation of transaction output numbers

While bitcoin normally uses VarInt to represent the number of transaction outputs that may be contained in a given transaction, it uses 32-bit unsigned integers to reference specific transaction output from other's transaction input. Even while with the current current block size limit the maximum number of transaction outputs that may be contained within a transaction is still below 2^16, we stick to the internal bitcoin transaction limits, which, in this case, is defined by transaction input structure (it's impossible to spend transaction output with index number >2^32).

## Reference implementation

[https://github.com/LNP-BP/rust-lnpbp/tree/develop/src/bp/seals](https://github.com/LNP-BP/rust-lnpbp/tree/develop/src/bp/seals)

## Acknowledgements

## References

## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.

## Test vectors
