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
Updated: 2022-12-23
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
-- number of decimal fractions (decimal numbers after floating point)
data Precision :: 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 |
                     14 | 15 | 16 | 17 | 18

data Outpoint :: txid [Byte ^ 32], vout U16

data PoR :: -- proof of reserves
    utxo Outpoint,
    proof [Bytes] -- auxilary data which are schema-specific

data Amount :: Zk64 -- asset amount

data Specification :: 
    ticker [Ascii ^ 1..8],
    name [Ascii ^ 1..40],
    details [Unicode ^ 40..256]?,
    precision Precision

interface RGB20
    -- Asset specification containing ticker, name, precision etc.
    global spec :: Specification

    -- Contract text is separated from the nominal since it must not be
    -- changeable by the issuer.
    global ricardianContract :: [Unicode]

    -- State which accumulates amounts issued
    global issuedSupply+ :: Amount
    -- State which accumulates amounts burned
    global burnedSupply* :: Amount
    -- State which accumulates amounts burned and then replaced
    global repalcedSupply* :: Amount

    -- Right to do a secondary (post-genesis) issue
    owned inflationAllowance* :: Amount
    -- Right to update asset Specification
    owned updateRight?
    -- Right to burn or replace existing assets
    owned burnRight?

    -- Ownership right over assets
    owned assetOwners* :: Amount
    
    -- Point for applying state extensions
    valency d10zedIssue

    -- !! means errors which may be returned
    genesis       -> spec, ricardianContract, 
                     issuedSupply, assetOwners+,
                     inflationAllowance*, 
                     updateRight?, burnRight?,
                     d10zedIssue?
                  !! supplyMismatch

    op transfer    :: previous assetOwners+, 
                   -> beneficiary assetOwners+
                   !! nonEqualAmounts

    -- question mark after `op` means optional operation, which may not be  
    -- provided by some of schemata implementing the intrface
    
    op? issue      :: used inflationAllowance+
                   -> issuedSupply, 
                      next inflationAllowance?,
                      beneficiary assetOwners*
                   !! supplyMismatch | issueExceedsAllowance

    op? d10zedIssue :: d10zedIssue, reserves PoR
                   -> beneficiary assetOwners+, issuedSupply
                   !! supplyMismatch | insufficientReserves | invalidProof(PoR)

    op? burn       :: used burnRight, proofs PoR*, burnedSupply
                   -> next burnRight?
                   !! supplyMismatch | invalidProof(PoR)
    
    op? replace    :: used burnRight, proofs POR*, replacedSupply
                   -> next burnRight?, beneficiary assetOwners+
                   !! nonEqualAmounts | supplyMismatch | invalidProof(PoR)

    op? rename     :: used updateRight
                   -> next UpdateRight?, new spec
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
