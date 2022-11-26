# List of specifications

LNP/BP stands for "Bitcoin Protocol / Lightning Network Protocol". This set of specifications covers standards & best practices for Layer 2, 3 solutions (and above) in cases when they do not require soft- or hard-forks on the Bitcoin blockchain level and are not directly related to issues covered in Lightning Network RFCs (BOLTs).

Basically, LNP/BPs cover everything that can be anchored to Bitcoin transactions, defines primitives for L2+ solution design and describes complex use cases which can be built from some primitives. This allows such solutions as financial assets, storage, messaging, computing and different forms of secondary markets leveraging Bitcoin security model and Bitcoin as a method of payment/medium of exchange.

Criteria for a LNP/BP specification proposal:

* Should not be covered by existing or proposed BIPs
* Should not cause soft- or hard-fork in Bitcoin blockchain (but may depend on soft-forks from an existing BIP proposals)
* Should not distort Bitcoin miner's economic incentives
* Should not pollute Bitcoin blockchain with unnecessary non-transaction related data or have to maintain such pollution as low as possible
* Must not require a utility or security tokens to function (but may enable creation of digital assets or tokenized physical goods)
* Must not depend on non-bitcoin blockchains (but may be applicable to other blockchains)

## List of current LNP/BP proposals

|                  No | Vertical            | Title                                                                                                                 | Authors                                  | Type          | Status   |
|--------------------:|---------------------|-----------------------------------------------------------------------------------------------------------------------|------------------------------------------|---------------|----------|
|  [1](lnpbp-0001.md) | Commitment schemes  | Key tweaking: collision-resistant elliptic curve-based commitments                                                    | Maxim Orlovsky et al                     | Standard      | Proposal |
|  [2](lnpbp-0002.md) | Commitment schemes  | Script tweaking: deterministic embedding of cryptographic commitments into script pubkeys                             | Maxim Orlovsky et al                     | Standard      | Proposal |
|  [3](lnpbp-0003.md) | Commitment schemes  | Deterministic definition of transaction output containing cryptographic commitment                                    | Giacomo Zucco et al                      | Standard      | Proposal |
|  [4](lnpbp-0004.md) | Commitment schemes  | Multi-protocol commitment scheme with zero-knowledge provable uniqueness                                              | Maxim Orlovsky                           | Standard      | Proposal |
|  [5](lnpbp-0005.md) | Wallet              | Universal short Bitcoin identifiers for blocks, transactions and their inputs & outputs                               | Christian Decker, Maxim Orlovsky         | Standard      | Proposal |
|  [6](lnpbp-0006.md) | Commitment schemes  | PayTweak: pay-to-contract deterministic bitcoin commitments                                                           | Maxim Orlovsky                           | Standard      | Draft    |
|                   7 | Commitment schemes  | Strict encoding and commitment-encode protocols                                                                       | Peter Todd, Maxim Orlovsky               | Standard      | Planned  |
|  [8](lnpbp-0008.md) | Commitment schemes  | Single-use-seals                                                                                                      | Peter Todd, Maxim Orlovsky               | Informational | Draft    |
|                 [9] | Commitment schemes  | Client-side-validation                                                                                                | Peter Todd, Maxim Orlovsky               | Informational | Draft    |
| [10](lnpbp-0010.md) | Commitment schemes  | Bitcoin transaction output-based single-use-seals                                                                     | Maxim Orlovsky et al                     | Standard      | Proposal |
| [11](lnpbp-0011.md) | Smart contracts     | RGB: Client-validated confidential smart contracts using bitcoin transaction graphs for Bitcoin and Lightning Network | Maxim Orlovsky et al                     | Informational | Proposal |
|                  12 | Smart contracts     | RGB Schema: client-side validation rules for RGB smart contracts                                                      | Maxim Orlovsky                           | Standard      | Planned  |
|                  13 | Smart contracts     | RGB client-side verification and data serialization                                                                   | Maxim Orlovsky                           | Standard      | Planned  |
|                  14 | Wallet              | Bech32 encoding for client-validated data                                                                             | Maxim Orlovsky                           | Standard      | Planned  |
| [15](lnpbp-0015.md) | Networking          | Handshake and encryption in network communications based on Noise\_XK (BOLT-8 extract)                                | Multiple peers                           | Standard      | Proposal |
|                  16 | Commitment schemes  | Anchoring multiple deterministic bitcoin commitments in the same transaction output                                   | Maxim Orlovsky                           | Standard      | Final    |
|                  17 | Commitment schemes  | TapRet: Taproot script tree-based OP_RETURN deterministic bitcoin commitments                                         | Maxim Orlovsky et al                     | Standard      | Final    |
| [18](lnpbp-0018.md) | Networking          | Native message framing protocol (BOLT-8 extract)                                                                      | Multiple peers                           | Standard      | Planned  |
|                  19 | Commitment schemes  | *Reserved for sign-to-contract deterministic bitcoin commitments*                                                     |                                          |               |          |
| [20](lnpbp-0020.md) | Wallet              | RGB fungible assets schema (RGB-20)                                                                                   | Multiple peers                           | Standard      | Final    |
| [21](lnpbp-0021.md) | Wallet              | RGB schema for NFTs representing ownership rights (RGB-21)                                                            | Maxim Orlovsky                           | Standard      | Proposal |
|                [22] | Wallet              | RGB reputation and identity schema (RGB-22)                                                                           | Maxim Orlovsky, Sabina Sachtachtinskagia | Standard      | Draft    |
|                  23 | Wallet              | RGB verifiable-unique history log for auditable data (RGB-23)                                                         | Maxim Orlovsky, Giacomo Zucco            | Standard      | Planned  |
|                  24 | Wallet              | RGB schema for decentralized global domain name system (RGB-24)                                                       | Maxim Orlovsky                           | Standard      | Planned  |
|                  30 | Wallet              | RGB schema for bitcoin-backed fungible assets with decentralized issuance (RGB-30)                                    | Maxim Orlovsky                           | Standard      | Planned  |
|           25-29, 31 | *Reserved*          | *For the future use by RGB schemata*                                                                                  |                                          |               |          |
|                  32 | Wallet              | BIP-32 derivation path extension for read-only wallets                                                                | Maxim Orlovsky                           | Standard      | Draft    |
|                [33] | Smart contracts     | Lightspeed: micro-payments for Lightning Network                                                                      | Maxim Orlovsky                           | Draft         |          |
|                [34] | Smart contracts     | Zero-knowledge arguments for data persistence using probabilistic checkable proofs                                    | Maxim Orlovsky                           | Standard      | Draft    |
|                [35] | Smart contracts     | Bifrost: LN message extensions for RGB data propagation                                                               | Maxim Orlovsky                           | Standard      | Planned  |
|                  36 | *Reserved*          | *For future use by lightning network protocol extensions*                                                             |                                          |               |          |
| [37](lnpbp-0037.md) | Wallet              | ~~Invoicing formats for RGB-20 fungible assets schema~~                                                               | Alekos Filini                            | Standard      | Rejected |
|                [38] | Wallet              | Universal LNP/BP invoices                                                                                             | Maxim Orlovsky                           | Standard      | Draft    |
|                  39 | *Reserved*          | *For future use by lightning network protocol extensions*                                                             |                                          |               |          |
|                [40] | Smart contracts     | Storm: trustless storage with escrow contracts                                                                        | Maxim Orlovsky                           | Standard      | Draft    |
|                  41 | Smart contracts     | Lightning network message extensions for Storm                                                                        | Maxim Orlovsky                           | Standard      | Planned  |
|                  42 | *Reserved*          | *For future use by lightning network protocol extensions*                                                             |                                          |               |          |
|                [43] | Wallet              | RGB-enabled BIP43 purpose field & identity system                                                                     | Maxim Orlovsky                           | Standard      | Draft    |
|                  44 | Wallet              | Script templating: BIP-32 & LNPBP-43 key derivations within for non-miniscript-compatible Bitcoin scripts             | Maxim Orlovsky                           | Standard      | Draft    |
|                  45 | Smart contracts     | Lightning network message extensions for decentralized exchange functionality                                         | Maxim Orlovsky                           | Standard      | Planned  |
|                  46 | Wallet              | Deterministic derivation paths for LNP                                                                                | Maxim Orlovsky                           | Draft         |          |
|               47,48 | *Reserved*          | *For future use by lightning network protocol extensions*                                                             |                                          |               |          |
|                  49 | Smart contracts     | Synchronized multi-hop state updates via delegation in Lightning network                                              | Maxim Orlovsky, Christian Decker         | Standard      | Planned  |
| [50](lnpbp-0050.md) | Smart contracts     | Bifrost: generalized Lightning network protocol core                                                                  | Maxim Orlovsky                           | Standard      | Planned  |
| [51](lnpbp-0051.md) | Smart contracts     | Bifrost: channel management protocol                                                                                  | Maxim Orlovsky                           | Standard      | Draft    |
|                  52 | Smart contracts     | Bifrost routed messaging system based on Sphix protocol                                                               | Maxim Orlovsky                           | Standard      | Draft    |
| [53](lnpbp-0053.md) | Smart contracts     | Milti-peer payment channels for Bifrost                                                                               | Maxim Orlovsky                           | Standard      | Draft    |
|                  54 | Smart contracts     | Channel factories based on Bifrost protocol                                                                           | Maxim Orlovsky                           | Standard      | Draft    |
| [55](lnpbp-0055.md) | Smart contracts     | HTLC channel synchronization in Bifrost                                                                               | Maxim Orlovsky                           | Standard      | Draft    |
|                  56 | Smart contracts     | PTLC channel synchronization in Bifrost                                                                               | Maxim Orlovsky                           | Standard      | Draft    |
|                  57 | Smart contracts     | Decentralized naming & name resolution system                                                                         | Maxim Orlovsky                           | Standard      | Planned  |
|                [58] | Commitment schemes  | Apophis: distributed elliptic curve-based key creation with shared secrets                                            | Maxim Orlovsky                           | Standard      | Draft    |
|                [59] | Smart contracts     | Typhon: trustless Bitcoin sidechains                                                                                  | Maxim Orlovsky                           | Standard      | Draft    |
|                [60] | Smart contracts     | Ibiss: incentive-based interactive anonymous settlement scheme for computation integrity arguments                    | Maxim Orlovsky, Sabina Sachtachtinskagia | Informational | Draft    |
|                [61] | Smart contracts     | Toth: incentive-based interactive settlement scheme for computation integrity arguments with reputation               | Maxim Orlovsky, Sabina Sachtachtinskagia | Informational | Draft    |
|                [62] | Smart contracts     | Prometheus: trustless multiparty computing with escrow & arbitration using Ibiss protocol on bitcoin blockchain       | Maxim Orlovsky                           | Standard      | Draft    |
|                  63 | Smart contracts     | Prometheus+: prometheus over LN with tokenized RGB reputation                                                         | Maxim Orlovsky                           | Standard      | Planned  |
|               64-79 | *Reserved*          | *For the future use by lightning network protocol extensions*                                                         |                                          |               |          |
|                [80] | Commitment schemes  | Merkle mountain ranges                                                                                                | Peter Todd                               | Standard      | Final    |
|                  81 | Commitment schemes  | Tagged merkle trees for client-side-validation                                                                        | Maxim Orlovsky, Peter Todd               | Standard      | Draft    |
|                  82 | Commitment schemes  | OpenTimestamps bitcoin transaction commitments                                                                        | Peter Todd                               | Standard      | Final    |
|                  83 | Commitment schemes  | OpenTimestamps proof construction & verification                                                                      | Peter Todd                               | Standard      | Final    |
|                  83 | Commitment schemes  | OpenTimestamps proof serialization                                                                                    | Peter Todd                               | Standard      | Final    |
|                  84 | Commitment schemes  | OpenTimestamps calendar and attestation services                                                                      | Peter Todd                               | Standard      | Final    |
|               85-89 | *Reserved*          |                                                                                                                       |                                          |               |          |
|                [90] | Smart contracts     | AluVM: virtual machine for client-side-validation                                                                     | Maxim Orlovsky                           | Standard      | Draft    |
|                  91 | Smart contracts     | AluVM extended instructions for handling RGB state validation                                                         | Maxim Orlovsky                           | Standard      | Planned  |
|               92-99 | *Reserved*          | *For future use by AluVM-specific standards*                                                                          |                                          |               |          |
|                 100 | *Reserved*          | *For future use by a scalable & confidential single-use-seal commitment layer 1*                                      |                                          |               |          |

### Invited or planned proposals to join LNP/BP standards family

1. Discreet log contracts:
   * [Specification effort](https://github.com/discreetlogcontracts/dlcspecs)
   * [Lightning Discreet Log Contract Channels](https://hackmd.io/@lpQxZaCeTG6OJZI3awxQPQ/LN-DLC)
2. Different [pre-Schnorr schemes for scriptless scripts](https://lists.linuxfoundation.org/pipermail/lightning-dev/2019-November/002316.html)
3. Other lightning network extensions: [eltoo](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2018-April/015907.html) and [PTLC](https://suredbits.com/payment-points-part-2-stuckless-payments/) proposals

## Verticals for LNP/BP proposals:

| Name               | Description                                               | Examples                                                                      |
|--------------------|-----------------------------------------------------------|-------------------------------------------------------------------------------|
| Commitment schemes | Commitment schemes used in client-side-validation         | Commitment schemes, zero knownledge                                           |
| Wallet             | Standards for wallet and other client-facing applications | Derivation paths, APIs, RGB asset schemata                                    |
| Networking         | P2P network communication protocols                       | Network encryption, framing, connectivity etc                                 |
| Smart contracts    | Distributed smart contract execution environment          | Scriptless scripts, RGB, lightning network applications, virtual machines etc |

[9]: https://diyhpl.us/wiki/transcripts/scalingbitcoin/milan/client-side-validation/
[22]: https://github.com/LNP-BP/LNPBPs/issues/29
[33]: https://github.com/LNP-BP/LNPBPs/issues/24
[34]: https://github.com/storm-org/storm-spec
[35]: https://github.com/LNP-BP/LNPBPs/pull/97
[38]: https://github.com/LNP-BP/FAQ/blob/master/Presentation%20slides/Universal%20LNP-BP%20invoices.pdf
[40]: https://github.com/storm-org/storm-spec
[43]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-February/018381.html
[58]: https://github.com/pandoracore/typhon-spec
[59]: https://github.com/pandoracore/typhon-spec
[60]: https://github.com/pandoracore/ibiss-spec
[61]: https://github.com/pandoracore/ibiss-spec
[62]: https://github.com/pandoracore/prometheus-spec
[80]: https://github.com/opentimestamps/opentimestamps-server/blob/master/doc/merkle-mountain-range.md
[90]: https://www.aluvm.org
