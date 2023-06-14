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
         Armando Dutra,
         Sabina Sachtachtinskagia,
         Martino Salvetti
Comments-URI: <https://github.com/LNP-BP/LNPBPs/discussions/140>
Status: Proposal
Type: Standards Track
Created: 2019-09-23
Updated: 2023-05-10
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

Overview

### Asset information

### Asset ownership

### Issue

- secondary issue
- issue delegation
- cancelling issue

### Burning & replace

### Proof of reserves


## Specification

Interface specification is the following Contractum code:

```haskell
-- Defined by LNPBP-31 standard in `RGBContract.sty` file
import level_decide_percent_6z2gZQEJsnP4xoNUC94vqYEE9V7gKQbeJhb5521xta5u as RGBContract

data Amount :: U64

interface RGB20
    -- Asset specification containing ticker, name, precision etc.
    global spec :: RGBTypes.DivisibleAssetSpec

    -- Contract text and creation date is separated from the spec since it must
    -- not be changeable by the issuer.
    global terms :: RGBTypes.RicardianContract
    global created :: RGBTypes.Timestamp

    -- State which accumulates amounts issued
    global issuedSupply+ :: Amount
    -- State which accumulates amounts burned
    global burnedSupply* :: Amount
    -- State which accumulates amounts burned and then replaced
    global replacedSupply* :: Amount

    -- Right to do a secondary (post-genesis) issue
    public inflationAllowance* :: Zk64
    -- Right to update asset Specification
    public updateRight?
    
    -- Right to open a new burn & replace epoch
    public burnEpoch?
    -- Right to burn or replace existing assets under some epoch
    public burnRight*

    -- Ownership right over assets
    private assetOwner* :: Zk64

    genesis       :: spec
                   , terms
                   , created
                   , issuedSupply
                   , reserves {RgbTypes.ProofOfReserves ^ 0..0xFFFF}
                  -> assetOwner*
                   , inflationAllowance*
                   , updateRight?
                   , burnEpoch?
                  -- errors which may be returned:
                  !! supplyMismatch
                   | invalidProof
                   | insufficientReserves

    op Transfer    :: previous assetOwner+ 
                   -> beneficiary assetOwner+
                   !! nonEqualAmounts

    -- question mark after `op` means optional operation, which may not be  
    -- provided by some of schemata implementing the interface
    
    op? Issue      :: used inflationAllowance+
                    , reserves {RgbTypes.ProofOfReserves ^ 0..0xFFFF}
                   -> issuedSupply
                    , future inflationAllowance*
                    , beneficiary assetOwner*
                   !! supplyMismatch
                    | invalidProof
                    | issueExceedsAllowance
                    | insufficientReserves

    op? OpenEpoch  :: used burnEpoch
                   -> next burnEpoch?
                    , burnRight

    op? Burn       :: used burnRight
                    , burnedSupply
                    , burnProofs {RgbTypes.ProofOfReserves ^ 0..0xFFFF}
                   -> future burnRight?
                   !! supplyMismatch 
                    | invalidProof
                    | insufficientCoverage
    
    op? Replace    :: used burnRight
                    , replacedSupply
                    , burnProofs {RgbTypes.ProofOfReserves ^ 0..0xFFFF}
                   -> future burnRight?
                    , beneficiary assetOwner+
                   !! nonEqualAmounts 
                    | supplyMismatch 
                    | invalidProof
                    | insufficientCoverage

    op? Rename     :: used updateRight
                   -> future updateRight?
                    , new spec
```

## Compatibility

This standard is the first fungible token interface defined for RGB assets,
so compatibility with other standards do not apply.

The standard provides a superset for ERC20 token capabilities; however due
to RGB nature it is not directly compatible with ERC20 and other similar
standards using account-based blockchains (and not client-side-validation
on UTXO chains). However, a ERC20 tokens may be re-issued or bridged under
this standard.


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
