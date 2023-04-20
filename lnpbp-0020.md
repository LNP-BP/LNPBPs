```
LNPBP: 0020
Aliases: RGB20
Vertical: Smart contracts
Title: RGB fungible assets interface (RGB-20)
Authors: Dr Maxim Orlovsky <orlovsky@lnp-bp.org>,
         Giacomo Zucco,
         Marco Amadori,
         Nicola Busanello,
         Federico Tenga,
         Sabina Sachtachtinskagia,
         Martino Salvetti
Comments-URI: <https://github.com/LNP-BP/LNPBPs/discussions/140>
Status: Proposal
Type: Standards Track
Created: 2019-09-23
Updated: 2023-04-20
Finalized: ~
Copyright: (0) public domain
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


## Abstract


## Background


## Motivation


## Design



## Specification

Interface specification is the following Contractum code:

```haskell
-- Defined by LNPBP-31 standard in `rgb.sty` file
import moment_shirt_uranium_E2hfuv3Za3kVo7MSCvSxC6uhYJEvkmZJ1NPz4g4oZWNw as RGBTypes

interface RGB20
    -- Asset specification containing ticker, name, precision etc.
    global spec :: RGBTypes.DivisibleAssetSpec

    -- Contract text is separated from the spec since it must not be
    -- changeable by the issuer.
    global terms :: RGBTypes.RicardianContract

    -- State which accumulates amounts issued
    global issuedSupply+ :: RGBTypes.Amount
    -- State which accumulates amounts burned
    global burnedSupply* :: RGBTypes.Amount
    -- State which accumulates amounts burned and then replaced
    global repalcedSupply* :: RGBTypes.Amount

    -- Right to do a secondary (post-genesis) issue
    owned inflationAllowance* :: RGBTypes.Amount
    -- Right to update asset Specification
    owned updateRight?
    -- Right to burn or replace existing assets
    owned burnRight?

    -- Ownership right over assets
    owned assetOwner+ :: RGBTypes.Amount

    -- !! means error symbols which may be returned
    genesis       -> spec
                   , terms
                   , reserves PoR*
                   , issuedSupply
                   , assetOwner+
                   , inflationAllowance*
                   , updateRight?
                   , burnRight?
                  !! supplyMismatch
                   | invalidProof(RGBTypes.PoR)
                   | insufficientReserves

    op Transfer    :: previous assetOwner+ 
                   -> beneficiary assetOwner+
                   !! nonEqualAmounts

    -- question mark after `op` means optional operation, which may not be  
    -- provided by some of schemata implementing the intrface
    
    op? Issue      :: used inflationAllowance+
                    , reserves PoR*
                   -> issuedSupply
                    , future inflationAllowance?
                    , beneficiary assetOwner*
                   !! supplyMismatch 
                    | issueExceedsAllowance
                    | insufficientReserves

    op? Burn       :: used burnRight
                    , burnProofs RGBTypes.PoR*
                    , burnedSupply
                   -> future burnRight?
                   !! supplyMismatch 
                    | invalidProof(RGBTypes.PoR)
                    | insufficientBurn
    
    op? Replace    :: used burnRight
                    , burnProofs RGBTypes.PoR*
                    , replacedSupply
                   -> future burnRight?
                    , beneficiary assetOwner+
                   !! nonEqualAmounts 
                    | supplyMismatch 
                    | invalidProof(RGBTypes.PoR)
                    | insufficientBurn

    op? Rename     :: used updateRight
                   -> future updateRight?
                    , new spec
```

## Compatibility


## Rationale

Include from
- <https://github.com/LNP-BP/LNPBPs/issues/27>
- <https://github.com/LNP-BP/LNPBPs/issues/28>
- <https://github.com/LNP-BP/LNPBPs/issues/50>

## Reference implementation

<https://github.com/RGB-WG/rgb-wallet/blob/master/std/src/interface/rgb20.rs>

## Acknowledgements


## References


## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.

<p xmlns:dct="http://purl.org/dc/terms/">
  <a rel="license"
     href="http://creativecommons.org/publicdomain/zero/1.0/">
    <img src="http://i.creativecommons.org/p/zero/1.0/88x31.png" style="border-style:none;" alt="CC0" />
  </a>
  <br />
  To the extent possible under law,
  <a rel="dct:publisher" href="https://lnp-bp.org">
    <span property="dcl:title">LNP/BP Standards Association</span></a>
  has waived all copyright and related or neighboring rights to this work.
</p>
