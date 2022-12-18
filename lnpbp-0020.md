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

-- allocation of assets to some transactou output
data Allocation :: TxOut, Amount

-- right over certain amount of assets; right owner may be unknown
data AssetControl :: TxOut?, Amount

data Nomination :: 
    ticker [Ascii ^ 1..8],
    name [Ascii ^ 1..40],
    details [Unicode ^ 40..256]?,
    contract [Unicode]??,
    precision DecFractions

data RGB20Info ::
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
    knownReissues [AssetControl], -- known future burn operations

    knownAllocations [Allocation] -- known asset amounts allocated to known UTXOs

interface RGB20 :: RGB20Info
    op transfer    :: inputs [TxOut ^ 1..] 
                   -> beneficiaries [Allocation]
                   !! -- options for operation failure:
                      outputSpent(TxOut)
                    | noAssets(TxOut)
                    | inequalAmounts

    -- question mark denotes optional operation, which may not be supported by 
    -- some of schemata implementing the intrface
    
    op? issue      :: usingRight TxOut, amount Amount 
                   -> nextRight TxOut?, beneficiaries [Allocation]
                   !! invalidRight
                    | invalidAmount

    op? dcntrlIssue -> reserves POR, beneficiaries [Allocation]
                   !! invalidReserves
                    | insufficientReserves

    op? burn       :: usingRight TxOut, proofs [POR], amount Amount
                   -> nextRight TxOut?
                   !! invalidRight
                    | invalidAmount
                    | invalidProof(POR)
    
    op? reissue    :: usingRight TxOut, proofs [POR], amount Amount
                   -> nextRight TxOut?, beneficiaries [Allocation]
                   !! invalidRight
                    | invalidAmount
                    | invalidProof(POR)

    op? renominate :: usingRight TxOut 
                   -> nextRight TxOut?, newNomination AssetInfo
                   !! invalidRight
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
