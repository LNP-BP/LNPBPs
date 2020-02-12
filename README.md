# LNP/BP Specifications

LNP/BP stands for "Bitcoin Protocol / Lightning Network Protocol". This set of specifications covers standards & best 
practices for Layer 2, 3 solutions (and above) in cases when they do not require soft- or hard-forks on the Bitcoin 
blockchain level and are not directly related to issues covered in Lightning Network RFCs (BOLTs).

Basically, LNP/BPs cover everything that can be anchored to Bitcoin transactions, defines primitives for L2+ solution
design and describes complex use cases which can be built from some primitives. This allows such solutions as financial 
assets, storage, messaging, computing and different forms of secondary markets leveraging Bitcoin security model and 
Bitcoin as a method of payment/medium of exchange.

Criteria for a LNP/BP specification proposal:
* Should not be covered by existing or proposed BIPs
* Should not cause soft- or hard-fork in Bitcoin blockchain (but may depend on soft-forks from an existing BIP proposals)
* Should not distort Bitcoin miner's economic incentives
* Should not pollute Bitcoin blockchain with unnecessary non-transaction related data or have to maintain such pollution
  as low as possible
* Should not be covered by existing or proposed BOLTs
* Must not require a utility or security tokens to function (but may enable creation of digital assets or tokenized 
physical goods)
* Must not depend on non-bitcoin blockchains (but may be applicable to bitcoin-compatible blockchains)


## Layers and fields for LNP/BP proposals:

Number (bottom-up) | Title | Description | Fields
------------------:| ----- | ----------- | -------
1                  | Transaction | Data or protocols defined for a single bitcoin transaction (both PSBT and mined) | Cryptographic primitives, Advanced multisignature schemes, Transaction structure
2                  | Transaction graph | Data and protocols defined on a transaction graph | State channels, Sidechains, Client-side validation
3                  | Client-validated data | Protocols and formats for off-chain data (persistent or ephemeral) | Cryptographic primitives, Commitments, Serialization, P2P
4                  | Client-validated graphs | Protocols and formats for off-chain graph structures | Cryptographic primitives, Commitments, Schemata, Serialization, P2P
5                  | Application | Specific high-level applications build of underlying layers (storage, messaging, assets etc) | Assets, Audit, Storage, Computing, Messaging, DEX/DMP*

* DEX: decentralized exchanges, DMP: decentralized markets/marketplaces

![LNP/BP Tech Stack structure](assets/lnpbp-layers.png)


## List of current LNP/BP proposals

Number | Layer | Field | Title | Owner | Type | Status
------:| ----- | ----- | ----- | ----- | ---- | ------
[1](lnpbp-0001.md) | Transaction (1) | Cryptographic primitives | Key tweaking: collision-resistant elliptic curve-based commitments | Maxim Orlovsky | Standard | Draft
[2](lnpbp-0002.md) | Transaction (1) | Cryptographic primitives | Deterministic embedding of elliptic curve-based commitments into transaction outputs | Maxim Orlovsky | Standard | Draft
[3](lnpbp-0003.md) | Transaction (1) | Cryptographic primitives | Deterministic definition of transaction output containing cryptographic commitment | Giacomo Zucco | Standard | Draft
[4](lnpbp-0004.md) | Transaction (1) | Cryptographic primitives | Multi-message commitment scheme with zero-knowledge provable unique properties | Maxim Orlovsky | Standard | Draft
[5](lnpbp-0005.md) | Transaction graph (2) | Client-side validation | Single-use seals with bitcoin transaction graph | Peter Todd | Concept | Draft
[6](lnpbp-0006.md) | Off-chain data (3) | Cryptographic primitives | Confidential amounts for client-validated data | Maxim Orlovsky | Standard | Draft
[7](lnpbp-0007.md) | Off-chain data (3) | Serialization | Types and requirements for off-chain data serialization | Maxim Orlovsky, Peter Todd | Best practices | Draft
[8](lnpbp-0008.md) | Off-chain data (3) | Serialization | Extra-transaction proof serialization for LNPBP-2 | Maxim Orlovsky | Standard | Draft
[9](lnpbp-0009.md) | Client-validated graphs (4) | Commitments | RGB: Client-validated rich state and smart contract system based on LNPBP1-8 standards | Maxim Orlovsky | Standard | Draft
[10](lnpbp-0010.md) | Client-validated graphs (4) | Serialization | Network serialization standards for RGB-related data structures | Maxim Orlovsky | Standard | Draft
[11](lnpbp-0011.md) | Client-validated graphs (4) | Schemata | RGB Schema: presets for RGB smart contracting and state transitions | Maxim Orlovsky | Standard | Draft
[12](lnpbp-0012.md) | Client-validated graphs (4) | Serialization | Network serialization standards for RGB schema | Maxim Orlovsky | Standard | Draft
[13](lnpbp-0013.md) | Transaction graph (2) | State channels | Prometheus: trustless multiparty computing with escrow & arbitration | Maxim Orlovsky | Standard | Draft
[14](lnpbp-0014.md) | Transaction graph (2) | State channels | Storm: trustless storage with escrow contracts | Maxim Orovsky | Standard | Draft
[15](lnpbp-0015.md) | Transaction (1) | Transaction structure | ECDSA-based discrete-log contract transactions |  | Standard | Draft


### List work-in-progress of LNP/BP proposals without an assigned standard number

Number | Layer | Field | Title | Owner | Type | Status
------:| ----- | ----- | ----- | ----- | ---- | ------
n/a    | Transaction graph (2) | State channels | Lightning network eltoo transaction structure | Christian Decker | Standard | Draft
n/a    | Transaction graph (2) | P2P | Pay-to-EC point multiparty transaction update coordination |  | Standard | Draft
n/a    | Client-validated graphs (4) | P2P | RGB state announcements for Lightning Network protocol | n/a | Standard | Draft
n/a    | Client-validated graphs (4) | P2P | RGB state updates over Lightning Network onion messaging | n/a | Standard | Draft
n/a    | Application (5) | DEX/DMP | Spectrum: routed RGB state swaps over Lightning Network | n/a | Standard | Draft
