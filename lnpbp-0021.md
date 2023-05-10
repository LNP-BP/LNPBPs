```
LNPBP: 0021
Vertical: Smart contracts
Title: RGB non-fungible assets interface for collectibles (RGB-21)
Authors: Dr Maxim Orlovsky <orlovsky@lnp-bp.org>,
         Hunter Trujillo,
         Federico Tenga,
         Zoe Faltibà,
         Carlos Roldan,
         Olga Ukolova,
         Giacomo Zucco,
         Armando Dutra
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

data ItemsCount :: U32

-- each collectible item is unique and must have an id
data TokenIndex :: U32

data OwnedFraction :: U64

-- allocation of a single token or its fraction to some transaction output
data Allocation :: TokenIndex, OwnedFraction

data EngravingData :: 
    appliedTo TokenIndex, 
    content EmbeddedMedia

data EmbeddedMedia ::
    type RGBTypes.MediaType,
    data [Byte]

data Attachment ::
    type RGBTypes.MediaType,
    digest: [U8 ^ 32] -- this can be any type of 32-byte hash, like SHA256(d), BLACKE3 etc

data TokenData ::
    id TokenIndex,
    ticker RGBTypes.Ticker?,
    name RGBTypes.Name?,
    details RGBTypes.Details?,
    -- always-embedded preview media < 64kb
    preview EmbeddedMedia?,
    -- external media which is the main media for the token
    media Attachment?,
    attachments { U8 -> ^ ..20 Attachment } -- auxiliary attachments by type (up to 20 attachments)
    -- output containing locked bitcoins; how reserves are proved is a matter
    -- of a specific schema implementation
    reserves RGBTypes.PoR? 

 -- each attachment type is a mapping from attachment id 
 -- (used as `Token.attachments` keys) to a short Ascii string
 -- verbally explaining the type of the attachment for the UI
 -- (like "sample" etc).
data AttachmentType :: 
    id U8, 
    description [Ascii ^ 1..20]

interface RGB21
    -- Asset specification containing ticker, name, precision etc.
    global spec :: RGBTypes.DivisibleAssetSpec
    
    -- Contract text is separated from the name since it must not be
    -- changeable by the issuer.
    global terms :: RGBTypes.RicardianContract

    -- Data for all issued tokens
    global tokens* :: TokenData
    global engravings* :: EngravingData
    global attachmentTypes* :: AttachmentType

    -- Right to do a secondary (post-genesis) issue
    owned inflationAllowance* :: ItemsCount
    -- Right to update asset name
    owned updateRight?

    -- Ownership right over assets
    owned assetOwner* :: Allocation

    genesis       -> name
                   , terms,
                   , reserves PoR*
                   , tokens*
                   , assetOwner*
                   , inflationAllowance*
                   , updateRight?
                   , attachmentType*
                  !! invalidProof(RGBTypes.PoR)
                   -- this error happens when amount of token > 1
                   | fractionOverflow(TokenIndex)
                   | insufficientReserves

    op Transfer    :: previous assetOwner+, 
                   -> beneficiaries assetOwner+
                   !! -- options for operation failure:
                      inequalFractions(TokenIndex)
                    | fractionOverflow(TokenIndex)
                    | nonFractionalToken

    op? TransferEngrave :: previous assetOwner+ 
                    , engraving  
                   -> beneficiaries assetOwner+
                   !! -- options for operation failure:
                      inequalValues(TokenIndex)
                    | nonFractionalToken
                    | nonEngravableToken

    op? Issue      :: used inflationAllowance
                    , newTokens tokens*
                    , reserves RGBTypes.PoR*
                    , newAttachmentTypes attachmentTypes*
                   -> future inflationAllowance
                    , beneficiaries assetOwners+
                   !! invalidReserves(RGBTypes.PoR)
                    | fractionOverflow(TokenIndex)
                    | insufficientAllowance
                    | insufficientReserves

    op? Rename     :: used updateRight
                   -> future updateRight?
                    , new name
```

## Compatibility


## Rationale


## Reference implementation


## Acknowledgements


## References


## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.
