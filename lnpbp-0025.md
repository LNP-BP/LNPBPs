```
LNPBP: 0025
Aliases: RGB25
Vertical: Smart contracts
Title: RGB collectible assets interface (RGB-25)
Authors: Zoe Faltibà,
         Nicola Busanello,
         Federico Tenga,
         Dr Maxim Orlovsky <orlovsky@lnp-bp.org>
Comments-URI: <https://github.com/LNP-BP/LNPBPs/discussions>
Status: Proposal
Type: Standards Track
Created: 2023-06-28
Updated: 2023-07-24
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
-- Defined by LNPBP-31 standard in `RGBContract.sty` file
import urn:ubideco:stl:6vbr9ZrtsD9aBjo5qRQ36QEZPVucqvRRjKCPqE8yPeJr#choice-little-boxer as RGBContract

interface RGB25
    global name :: RGBContract.Name
    global details :: RGBContract.Details
    global precision :: RGBContract.Precision

    global data :: RGBContract.ContractData
    global created :: RGBContract.Timestamp

    -- State which contains amounts issued
    global issuedSupply :: RGBContract.Amount
    -- State which accumulates amounts burned
    global burnedSupply* :: RGBContract.Amount

    -- Right to burn or replace existing assets under some epoch
    public burnRight*

    -- Ownership right over assets
    private assetOwner+ :: Zk64

    genesis       :: name
                   , details
                   , precision
                   , data
                   , created
                   , issuedSupply
                   , reserves {RGBContract.ProofOfReserves ^ 0..0xFFFF}
                  -> assetOwner+
                  -- errors which may be returned:
                  !! supplyMismatch
                   | invalidProof
                   | insufficientReserves

    op Transfer    :: previous assetOwner+
                   -> beneficiary assetOwner+
                   !! nonEqualAmounts

    op? Burn       :: used burnRight
                    , burnedSupply
                    , burnProofs {RGBContract.ProofOfReserves ^ 0..0xFFFF}
                   -> future burnRight?
                   !! supplyMismatch
                    | invalidProof
                    | insufficientCoverage
```

## Compatibility

This standard is a collectible token interface defined for RGB assets,
so compatibility with other standards do not apply.

## Rationale

## Reference implementation

<https://github.com/RGB-WG/rgb-std/blob/master/src/interface/rgb25.rs>


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
