```
LNPBP: 0030
Aliases: RGB30
Vertical: Smart contracts
Title: Interface for fungible RGB assets with decentralized issue (RGB-30)
Authors: Dr Maxim Orlovsky <orlovsky@lnp-bp.org>
Comments-URI: <https://github.com/LNP-BP/LNPBPs/discussions/140>
Status: Proposal
Type: Standards Track
Created: 2021-06-23
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
using StandardTypes -- see LNPBP-31 standard

interface RGB30
    -- Asset specification containing ticker, name, precision etc.
    global spec :: Specification

    -- Contract text is separated from the nominal since it must not be
    -- changeable by the issuer.
    global ricardianContract :: [Unicode]

    -- Ownership right over assets
    owned assetOwners* :: Amount
    
    -- Point for applying state extensions
    valency issueRight

    genesis       -> spec, ricardianContract, issueRight

    op transfer    :: previous assetOwners+, 
                   -> beneficiary assetOwners+
                   !! nonEqualAmounts

    op pegIn       :: issueRight, reserves PoR
                   -> beneficiary assetOwners+, issuedSupply
                   !! supplyMismatch | insufficientReserves | invalidProof(PoR)
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
