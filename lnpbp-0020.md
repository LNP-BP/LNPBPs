```
LNPBP: 0020
Vertical: Smart contracts
Title: RGB fungible assets interface (RGB-20)
Authors: Dr Maxim Orlovsky <orlovsky@protonmail.ch>,
         Giacomo Zucco,
         Marco Amadori,
         Nicola Busanello,
         Federico Tenga,
         Sabina Sachtachtinskagia,
         Martino Salvetti
Comments-URI: <https://github.com/LNP-BP/LNPBPs/issues/70>
Status: Proposal
Type: Standards Track
Created: 2019-09-23
Finalized: 2022-12-18
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
-- # Defining main data structures

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

interface RGB20 :: RGB20Info
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

    owned IssueRight+ :: Amount
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
```

## Compatibility


## Rationale

Include from
- https://github.com/LNP-BP/LNPBPs/issues/27
- https://github.com/LNP-BP/LNPBPs/issues/28
- https://github.com/LNP-BP/LNPBPs/issues/50

## Reference implementation


## Acknowledgements


## References


## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.
