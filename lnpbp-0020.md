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
data DecFractions :: 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 |
                     14 | 15 | 16 | 17 | 18

data TxOut :: txid [Byte ^ 32], vout U16

data POR :: -- proof of reserves
    utxo TxOut,
    proof [Bytes] -- auxilary data which are schema-specific

data Amount :: U64 -- asset amount

data Denomination :: 
    ticker [Ascii ^ 1..8],
    name [Ascii ^ 1..40],
    details [Unicode ^ 40..256]?,
    precision DecFractions

interface RGB20
    global Denomination :: Denomination

    -- Contract text is separated from the denomination since it must not be
    -- changeable by an issuer.
    global Contract :: [Unicode]
    -- The difference between `global _ :: [_]` and `global _? :: _` is that
    -- the first indicates that the global state may not be present, while
    -- the second requires the state must be present and have a form of array
    -- of the elements.

    -- state which accumulates amounts issued
    global Issued* :: Amount
    -- state which accumulates amounts burned
    global Burned* :: Amount
    -- state which accumulates amounts burned and then replaced
    global Repalced* :: Amount

    owned IssueRight* :: Amount
    owned DenominationRight?
    owned BurnRight?

    owned Assets* :: Amount

    -- operations are declared as
    -- `op` NAME `::` CLOSED_SEALS,+ INPUT_METADATA,*
    --           `->` DEFINED_SEALS,+
    --           `<-` NEW_GLOBAL_STATE,*
    --           `!!` ERRORS|*

    op transfer    :: inputs [Assets] 
                   -> beneficiaries [Assets]
                   !! inequalAmounts

    -- question mark denotes optional operation, which may not be supported by 
    -- some of schemata implementing the intrface
    
    op? issue      :: using IssueRight
                   -> next IssueRight?, beneficiaries [Assets]
                   <- amount Issued
                   !! nonEqualAmounts

    op? dcntrlIssue -> reserves POR, beneficiaries [Assets]
                   <- amount Issued
                   !! invalidReserves
                    | insufficientReserves

    op? burn       :: using BurnRight, proofs [POR], amount Amount
                   -> next BurnRight?
                   <- amount Burned
                   !! nonEqualAmounts
                    | invalidProof(POR)
    
    op? replace    :: using BurnRight, proofs [POR]
                   -> next BurnRight?, beneficiaries [Assets]
                   <- amount Replaced
                   !! nonEqualAmounts
                    | invalidProof(POR)

    op? rename     :: using DenominationRight
                   -> next DenominationRight?, new Denomination

    op? decentralizedIssue -> beneficiaries [Assets]
                           <- proofs [POR]
                           !! invalidProof(POR)
```

## Compatibility


## Rationale

Include from
- https://github.com/LNP-BP/LNPBPs/issues/27
- https://github.com/LNP-BP/LNPBPs/issues/28
- https://github.com/LNP-BP/LNPBPs/issues/50

## Reference implementation

Simple asset issuance:

```Haskell
import RGB20

schema SimpleAsset
    global Denomination :: Denomination
    global Contract :: [Unicode]

    global Issued* :: Amount
    owned Assets* :: Amount

    genesis -> allocations [Assets]
            <- totalAmount Issued, naming Denomination, contract Contract
        sum allocations =? totalAmount !! nonEqualAmounts

    op transfer :: inputs [Assets] -> beneficiaries [Assets]
        sum inputs =? sum beneficiaries !! nonEqualAmounts

implement RGB20 for SimpleAsset
    -- all names match interface so no explicit declarations here are needed

let issuerOwned := Seal fac503c4641c3deda72a2d00bc9d6ff1094b15276c386efea403746a91436772, 1

contract sampleAsset := SimpleAsset (
    naming := Denomination(
        ticker := "OTI",
        name := "One time issued token",
        details := "Absolutely useless",
        precision := 8
    ),
    contract := """
        THE ASSET TOKEN IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
        EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
        MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
        IN NO EVENT SHALL THE AUTHORS BE LIABLE FOR ANY CLAIM, DAMAGES OR
        OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE,
        ARISING FROM, OUT OF OR IN CONNECTION WITH THE ASSET TOKEN OR THE USE
        OR OTHER DEALINGS IN THE CONTRACT.
        """,
    allocations := [
        (issuerOwned, 10_000_000__0000_0000)
    ],
    totalAmount := 10_000_000__0000_0000
)

let friendOwned := Seal ~, 0
let issuerChange := Seal ~, 1

transition sendToFriend := SimpleAsset.transfer (
    inputs := [issuerOwned],
    beneficiaries := [
        (friendOwned, 1_000_000__0000_0000),
        (issuerChange, 9_000_000__0000_0000)
    ]
)
```


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
