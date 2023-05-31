# LNPBPs: LNP/BP Standards Repository

LNP/BP stands for "Bitcoin Protocol / Lightning Network Protocol". This 
repository covers standards & best practices for Layer 2+ in cases when they do
not require soft- or hard-forks of the Bitcoin blockchain level and are not 
directly related to issues covered in Lightning Network RFCs (BOLTs).

Basically, LNP/BPs cover everything that can be anchored to Bitcoin 
transactions, defines primitives for L2+ solution design and describes complex 
use cases which can be built from some primitives. This allows such solutions as
financial assets, storage, messaging, computing and different forms of secondary
markets leveraging Bitcoin security model and Bitcoin as a method of 
payment/medium of exchange.

Criteria for a LNP/BP specification proposal:

* Should not be covered by existing or proposed BIPs
* Should not cause soft- or hard-fork in Bitcoin blockchain (but may depend on 
  soft-forks from an existing BIP proposals)
* Should not distort Bitcoin miner's economic incentives
* Should not pollute Bitcoin blockchain with unnecessary non-transaction related
  data or have to maintain such pollution as low as possible
* Must not require a utility or security tokens to function (but may enable
  creation of digital assets or tokenized physical goods)
* Must not depend on non-bitcoin blockchains (but may be applicable to other 
  blockchains)


## Verticals for LNP/BP proposals:

| Name            | Description                   | Examples                                        |
|-----------------|-------------------------------|-------------------------------------------------|
| Cryptography    | Cryptographic primitives      | Cryptography, zero knowledge                    |
| Wallet          | Standards for wallet and apps | Derivation paths, APIs, RGB asset schemata      |
| Networking      | P2P network communications    | Network encryption, framing, connectivity etc   |
| Smart contracts | Distributed smart contracts   | Scriptless scripts, RGB, lightning channels etc |


## List of LNP/BP standards and proposals

|    No | Vertical        | Title                                                                                     | Type          | Status   |
|------:|-----------------|-------------------------------------------------------------------------------------------|---------------|----------|
|   [1] | Cryptography    | Key tweaking: collision-resistant elliptic curve-based commitments                        | Standard      | Proposal |
|   [2] | Cryptography    | Script tweaking: deterministic embedding of cryptographic commitments into script pubkeys | Standard      | Proposal |
|   [3] | Cryptography    | Deterministic definition of transaction output containing cryptographic commitment        | Standard      | Proposal |
|   [4] | Cryptography    | Multi-protocol commitment scheme with zero-knowledge provable uniqueness                  | Standard      | Proposal |
|   [5] | Wallet          | Universal short Bitcoin identifiers for blocks, transactions and their inputs & outputs   | Standard      | Proposal |
|   [6] | Cryptography    | ~~PayTweak: pay-to-contract deterministic bitcoin commitments~~                           | Standard      | Rejected |
|     7 | Cryptography    | Commitments for structural and hierarchical data                                          | Standard      | Proposal |
|   [8] | Cryptography    | Single-use-seals                                                                          | Informational | Draft    |
|   [9] | Cryptography    | Client-side-validation                                                                    | Informational | Draft    |
|  [10] | Cryptography    | Bitcoin transaction output-based single-use-seals                                         | Standard      | Proposal |
|    11 | Cryptography    | Anchoring multiple deterministic bitcoin commitments in the same transaction output       | Standard      | Final    |
|  [12] | Cryptography    | TapRet: Taproot script tree-based OP_RETURN deterministic bitcoin commitments             | Standard      | Final    |
|  [13] | Smart contracts | RGB: Client-side-validated confidential smart contracts for Bitcoin and Lightning Network | Informational | Draft    |
|    14 | Smart contracts | RGB Schema: client-side validation rules for RGB smart contracts                          | Standard      | Planned  |
|    15 | Smart contracts | RGB client-side verification and data serialization                                       | Standard      | Planned  |
|    16 | Smart contracts | AluVM instruction set architecture extensions for handling RGB state validation           | Standard      | Planned  |
| 17-18 | *Reserved*      | *For the future use by RGB extensions*                                                    |               |          |
|    19 | Cryptography    | *Reserved for sign-to-contract deterministic bitcoin commitments*                         |               |          |
|  [20] | Wallet          | RGB fungible assets interface (RGB-20)                                                    | Standard      | Final    |
|  [21] | Wallet          | RGB non-fungible collectibles interface (RGB-21)                                          | Standard      | Proposal |
|  [22] | Wallet          | RGB reputation and identity interface (RGB-22)                                            | Standard      | Draft    |
|  [23] | Wallet          | RGB verifiable-unique history log for auditable data (RGB-23)                             | Standard      | Planned  |
|  [24] | Wallet          | RGB schema for decentralized global domain name system (RGB-24)                           | Standard      | Planned  |
| 25-29 | *Reserved*      | *For the future use by RGB schemata*                                                      |               |          |
|  [30] | Wallet          | Interface for fungible RGB assets with decentralized issue (RGB-30)                       | Standard      | Final    |
|  [31] | *Reserved*      | Standard Contractum Libraries (SCL)                                                       | Standard      | Draft    |
|    32 | Wallet          | BIP-32 derivation path extension for read-only wallets                                    | Standard      | Draft    |
|  [33] | Smart contracts | Lightspeed: micro-payments for Lightning Network                                          | Draft         |          |
|  [34] | Smart contracts | Zero-knowledge arguments for data persistence using probabilistic checkable proofs        | Standard      | Draft    |
|  [35] | Smart contracts | Bifrost: LN message extensions for RGB data propagation                                   | Standard      | Planned  |
|    36 | *Reserved*      | *For future use by bitcoin protocol extensions*                                           |               |          |
|  [37] | Wallet          | ~~Invoicing formats for RGB-20 fungible assets schema~~                                   | Standard      | Rejected |
|  [38] | Wallet          | Universal LNP/BP invoices                                                                 | Standard      | Draft    |
|    39 | *Reserved*      | *For future use by bitcoin protocol extensions*                                           |               |          |
|  [40] | Smart contracts | Storm: trustless storage with escrow contracts                                            | Standard      | Draft    |
|    41 | Smart contracts | Lightning network message extensions for Storm                                            | Standard      | Planned  |
|    42 | *Reserved*      | *For future use by lightning network protocol extensions*                                 |               |          |
|  [43] | Wallet          | RGB-enabled BIP43 purpose field & identity system                                         | Standard      | Draft    |
|    44 | Wallet          | Script templating: key derivations within Bitcoin scripts                                 | Standard      | Planned  |
|    45 | Smart contracts | Lightning network message extensions for decentralized exchange functionality             | Standard      | Planned  |
|    46 | Wallet          | Deterministic derivation paths for LNP                                                    | Draft         |          |
| 47,48 | *Reserved*      | *For future use by bitcoin protocol extensions*                                           |               |          |
|    49 | Smart contracts | Synchronized multi-hop state updates via delegation in Lightning network                  | Standard      | Planned  |
|  [50] | Smart contracts | Bifrost: generalized Lightning network protocol core                                      | Standard      | Planned  |
|  [51] | Smart contracts | Bifrost: channel management protocol                                                      | Standard      | Draft    |
|    52 | Smart contracts | Bifrost routed messaging system based on Sphinx protocol                                  | Standard      | Draft    |
|  [53] | Smart contracts | Milti-peer payment channels for Bifrost                                                   | Standard      | Draft    |
|    54 | Smart contracts | Channel factories based on Bifrost protocol                                               | Standard      | Draft    |
|  [55] | Smart contracts | HTLC channel synchronization in Bifrost                                                   | Standard      | Draft    |
|    56 | Smart contracts | PTLC channel synchronization in Bifrost                                                   | Standard      | Draft    |
|    57 | Smart contracts | Decentralized naming & name resolution system                                             | Standard      | Planned  |
|  [58] | Cryptography    | Apophis: distributed elliptic curve-based key creation with shared secrets                | Standard      | Draft    |
|  [59] | Smart contracts | Typhon: trustless Bitcoin sidechains                                                      | Standard      | Draft    |
|  [60] | Smart contracts | Ibiss: reputation-based interactive computation integrity arguments                       | Informational | Draft    |
|  [61] | Smart contracts | Toth: reputation-based interactive settlement for computation integrity arguments         | Informational | Draft    |
|  [62] | Smart contracts | Prometheus: trustless multiparty computing with escrow & arbitration on bitcoin           | Standard      | Draft    |
|    63 | Smart contracts | Prometheus+: prometheus over LN with tokenized RGB reputation                             | Standard      | Planned  |
| 64-79 | *Reserved*      | *For the future use by lightning network protocol extensions*                             |               |          |
|  [80] | Cryptography    | Merkle mountain ranges                                                                    | Standard      | Final    |
|    81 | Cryptography    | Tagged merkle trees for client-side-validation                                            | Standard      | Draft    |
|    82 | Cryptography    | OpenTimestamps bitcoin transaction commitments                                            | Standard      | Final    |
|    83 | Cryptography    | OpenTimestamps proof construction & verification                                          | Standard      | Final    |
|    83 | Cryptography    | OpenTimestamps proof serialization                                                        | Standard      | Final    |
|    84 | Cryptography    | OpenTimestamps calendar and attestation services                                          | Standard      | Final    |
| 85-99 | *Reserved*      |                                                                                           |               |          |
|   100 | *Reserved*      | *For future use by a scalable & confidential single-use-seal commitment layer 1*          |               |          |

### Invited or planned proposals to join LNP/BP standards family

1. Discreet log contracts:
   * [Specification effort][dlc]
   * [Lightning Discreet Log Contract Channels][dlc-ln]
2. Different [pre-Schnorr schemes for scriptless scripts][scriptless]
3. Other lightning network extensions: 
   * [eltoo]
   * [PTLC]

[1]: lnpbp-0001.md
[2]: lnpbp-0002.md
[3]: lnpbp-0003.md
[4]: lnpbp-0004.md
[5]: lnpbp-0005.md
[6]: lnpbp-0006.md
[7]: lnpbp-0007.md
[8]: lnpbp-0008.md
[9]: https://diyhpl.us/wiki/transcripts/scalingbitcoin/milan/client-side-validation/

[10]: lnpbp-0010.md
[11]: lnpbp-0011.md
[12]: lnpbp-0012.md
[13]: lnpbp-0013.md
[14]: lnpbp-0014.md
[15]: lnpbp-0015.md
[16]: lnpbp-0016.md
[17]: lnpbp-0017.md
[18]: lnpbp-0018.md
[19]: lnpbp-0019.md

[20]: lnpbp-0020.md
[21]: lnpbp-0021.md
[22]: lnpbp-0022.md
[23]: lnpbp-0023.md
[24]: lnpbp-0024.md
[25]: lnpbp-0025.md
[26]: lnpbp-0026.md
[27]: lnpbp-0027.md
[28]: lnpbp-0028.md
[29]: lnpbp-0029.md

[30]: lnpbp-0030.md
[31]: lnpbp-0031.md
[32]: lnpbp-0032.md
[33]: https://github.com/LNP-BP/LNPBPs/issues/24
[34]: https://github.com/storm-org/storm-spec
[35]: https://github.com/LNP-BP/LNPBPs/pull/97
[36]: lnpbp-0036.md
[37]: lnpbp-0037.md
[38]: https://github.com/LNP-BP/FAQ/blob/master/Presentation%20slides/Universal%20LNP-BP%20invoices.pdf
[39]: lnpbp-0039.md

[40]: https://github.com/storm-org/storm-spec
[41]: lnpbp-0041.md
[42]: lnpbp-0042.md
[43]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-February/018381.html
[44]: lnpbp-0044.md
[45]: lnpbp-0045.md
[46]: lnpbp-0046.md
[47]: lnpbp-0047.md
[48]: lnpbp-0048.md
[49]: lnpbp-0049.md

[50]: lnpbp-0050.md
[51]: lnpbp-0051.md
[52]: lnpbp-0052.md
[53]: lnpbp-0053.md
[54]: lnpbp-0054.md
[55]: lnpbp-0055.md
[56]: lnpbp-0056.md
[57]: lnpbp-0057.md
[58]: https://github.com/pandoracore/typhon-spec
[59]: https://github.com/pandoracore/typhon-spec

[60]: https://github.com/pandoracore/ibiss-spec
[61]: https://github.com/pandoracore/ibiss-spec
[62]: https://github.com/pandoracore/prometheus-spec
[63]: lnpbp-0063.md
[64]: lnpbp-0064.md
[65]: lnpbp-0065.md
[66]: lnpbp-0066.md
[67]: lnpbp-0067.md
[68]: lnpbp-0068.md
[69]: lnpbp-0069.md

[70]: lnpbp-0070.md
[71]: lnpbp-0071.md
[72]: lnpbp-0072.md
[73]: lnpbp-0073.md
[74]: lnpbp-0074.md
[75]: lnpbp-0075.md
[76]: lnpbp-0076.md
[77]: lnpbp-0077.md
[78]: lnpbp-0078.md
[79]: lnpbp-0079.md

[80]: https://github.com/opentimestamps/opentimestamps-server/blob/master/doc/merkle-mountain-range.md
[81]: lnpbp-0081.md
[82]: lnpbp-0082.md
[83]: lnpbp-0083.md
[84]: lnpbp-0084.md
[85]: lnpbp-0085.md
[86]: lnpbp-0086.md
[87]: lnpbp-0087.md
[88]: lnpbp-0088.md
[89]: lnpbp-0089.md

[90]: lnpbp-0090.md
[91]: lnpbp-0091.md
[92]: lnpbp-0092.md
[93]: lnpbp-0093.md
[94]: lnpbp-0094.md
[95]: lnpbp-0095.md
[96]: lnpbp-0096.md
[97]: lnpbp-0097.md
[98]: lnpbp-0098.md
[99]: lnpbp-0099.md

[100]: lnpbp-0100.md

[dlc]: https://github.com/discreetlogcontracts/dlcspecs
[dlc-ln]: https://hackmd.io/@lpQxZaCeTG6OJZI3awxQPQ/LN-DLC
[scriptless]: https://lists.linuxfoundation.org/pipermail/lightning-dev/2019-November/002316.html
[eltoo]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2018-April/015907.html
[PTLC]: https://suredbits.com/payment-points-part-2-stuckless-payments/
