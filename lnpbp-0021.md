```
LNPBP: 0021
Vertical: Smart contracts
Title: RGB non-fungible assets interface for collectibles (RGB-21)
Authors: Dr Maxim Orlovsky <orlovsky@lnp-bp.org>,
         Hunter Trello,
         Federico Tenga,
         Carlos Roldan,
         Olgsa Ukolova,
         Giacomo Zucco,
Comments-URI: <https://github.com/LNP-BP/LNPBPs/issues/70>
Status: Proposal
Type: Standards Track
Created: 2020-09-10
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


## Background


## Motivation


## Design

- [x] Media as URI (which can be attachments distributed with Storm or usual URLs)
- [x] Small media ("previews" if given together with large media)
- [x] Fraction of assets
- [x] Engravings
- [x] Reserves
- [x] Possible decentralized issue


## Specification

Interface specification is the following Contractum code:

```haskell
-- # Defining main data structures

-- collectibles are usually scarse, so we limit their max number to 64k
data ItemsCount :: U16

-- each collectible item is unique and must have an id
data TokenId :: U16

-- Collectibles are owned in fractions of a single item; here the owned
-- fraction means `1/F`. Ownership of the full item is specified `F=1`;
-- half of an item as `F=2` etc.
data OwnedFraction :: U64

data TxOut :: txid [Byte ^ 32], vout U16

-- allocation of a single token or its fraction to some transaction output
data Allocation :: TxOut, TokenId, OwnedFraction

-- right over certain amount of collectible items; right owner may be unknown
data AssetControl :: TxOut?, ItemsCount

data Media ::
    type MimeType,
    data [Byte]

data Attachment ::
    type MimeType,
    uri: Uri

data POR :: -- proof of reserves
    reserves TxOut,
    proof [Bytes] -- schema-specific proof 

data Token ::
    id Id,
    name [Ascii ^ 1..40],
    details [Unicode ^ 40..256]?,
    media Media?,
    attachment Attachment?,
    reserves: POR? -- output containing locked bitcoins; how reserves are
                   -- proved is a matter of a specific schema implementation

data Nomination :: 
    ticker [Ascii ^ 1..8],
    name [Ascii ^ 1..40],
    details [Unicode ^ 40..256]?,
    contract [Unicode]??,
    isFractional: Bool

data CollectionInfo ::
    knownInfo Nomination,
    origInfo Nomination,
    isFinalInfo Bool,
    renominationRight TxOut?,

    isSupplyKnown Bool, -- indicates that all issues, burn and reissues are known
    supplyKnown Amount, -- returns information about known cirtulating supply
    supplyLimit Amount, -- maximum possible asset inflation
    pastIssues [AssetControl], -- known past issue operations
    futureIssues [AssetControl], -- known future issue operations
    knownBurns [AssetControl], -- known past burn operations
    knownReissues [AssetControl], -- known past reissue operations

    tokens [(Token)]
    -- known asset amounts allocated to known UTXOs
    knownAllocations [(Allocation, [Engraving])]

interface RGB21 :: CollectionInfo
    op transfer    :: inputs [TxOut ^ 1..] 
                   -> beneficiaries [Allocation]
                   !! -- options for operation failure:
                      outputSpent(TxOut)
                    | noTokens(TxOut)
                    | inequalFractions(TokenId)
                    | nonFractionalToken

    -- question mark denotes optional operation, which may not be supported by 
    -- some of schemata implementing the intrface

    op? engrave    :: inputs [TxOut ^ 1..] 
                   -> beneficiaries [(Allocation, Engraving?)]
                   !! outputSpent(TxOut)
                    | noTokens(TxOut)
                    | inequalFractions(TokenId)
                    | nonFractionalToken
                    | noEngravings

    op? issue      :: usingRight TxOut, 
                   -> nextRight TxOut?, tokens [Token], beneficiaries [Allocation]
                   !! invalidRight
                    | invalidReserves(POR)

    -- decentralized issue
    op? dcntrlIssue -> tokens [Token], 
                       beneficiaries [Allocation]
                    !! noReserves(TokenId)
                     | invalidReserves(POR)

    op? renominate :: usingRight TxOut 
                   -> nextRight TxOut?, newNomination AssetInfo
                   !! invalidRight
```

## Compatibility


## Rationale


## Reference implementation


## Acknowledgements


## References


## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.
