```
LNPBP: 0031
Vertical: Smart contracts
Title: Standard Contractum Libraries (SCL)
Authors: Dr Maxim Orlovsky <orlovsky@lnp-bp.org>
Comments-URI: <https://github.com/LNP-BP/LNPBPs/discussions/141>
Status: Proposal
Type: Standards Track
Created: 2022-12-23
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

The specification is the actual Standard Contractum Library code:

`rgb.sty`:
```haskell
-- number of decimal fractions (decimal numbers after floating point)
data Precision :: indivisible:0 
                | deci:1 
                | centi:2 
                | milli:3
                | deciMilli:4
                | centiMilli:5 
                | micro:6 
                | deciMicro:7 
                | centiMicro:8 
                | nano:9 
                | deciNano:10 
                | centiNano:11 
                | pico:12 
                | deciPico:13 
                | centiPico:14 
                | femto:15 
                | deciFemto:16 
                | centiFemto:17 
                | atto:18

data Outpoint :: txid [Byte ^ 32], vout U16

data PoR :: -- proof of reserves
    utxo Outpoint,
    proof [Bytes] -- auxilary data which are schema-specific

data Amount :: Zk64 -- asset amount

data AssetNaming ::
    ticker [Ascii ^ 1..8],
    name [Ascii ^ 1..40],
    details [Unicode ^ 40..256]?,

data DivisibleAssetSpec ::
    naming AssetNaming, 
    precision Precision
    
data RicardianContract :: [Unicode]
```

`collections.con`:
```haskell
mod Collections

fn sum col, el :: col -> el
    sum [Num], Num := 
        | [s:xs] -> s + sum xs
        | [s:] -> s

fn count col, c :: col -> c
    count [Any ^ ..0xFF], U8 :=
        | [s:xs] -> 1 + count xs
        | [s:] -> 1
        | [] -> 0
    count [Any ^ ..0xFFFF], U16 :=
        | [s:xs] -> 1 + count xs
        | [s:] -> 1
        | [] -> 0
```

`bp.con`:
```haskell
mod BP

data Sats :: U64
data Height :: U32
data Timestamp :: I64
data Difficulty: U32

data Txid :: [Byte ^ 32]
data Outpoint :: txid Txid, vout U16
data Seal :: txid Txid?, vout U16
data Assignment t :: (Seal, t)

data TxStatus :: unknown | mempool | height Height
data LockTime :: height Height | blocks U32

-- Public key variants
data FullPubkey :: 04, x [U8 ^ 32], y [U8 ^ 32]
data PubkeyParity :: even=02 | odd=03
data CompressedPubkey :: parity PubkeyParity, x [U8 ^ 32]
data Pubkey :: uncompressed FullPubkey | compressed ShortPubkey
data XonlyPubkey :: [U8 ^ 32]

-- Hash types

-- Scripts
data Script :: [Instruction]
data WitnessProgram :: [Byte]

data SegwitVer :: segwit=0 | taproot=0x51 |
    ver2=0x52 | ver3=0x53 | ver4=0x54 | ver5=0x55 | ver6=0x56 | ver7=0x57 |
    ver8=0x58 | ver9=0x59 | ver10=0x5A | ver11=0x5B | ver12=0x5C | ver13=0x5D |
    ver14=0x5E | ver15=0x5F | ver16=0x60
data FutureSegwitVer :: 
    ver2=0x52 | ver3=0x53 | ver4=0x54 | ver5=0x55 | ver6=0x56 | ver7=0x57 |
    ver8=0x58 | ver9=0x59 | ver10=0x5A | ver11=0x5B | ver12=0x5C | ver13=0x5D |
    ver14=0x5E | ver15=0x5F | ver16=0x60

data ScriptPubkey ::
    bare Script |
    pk Pubkey |
    pkh PubkeyHash |
    sh ScriptHash |
    wpkh WPubkeyHash |
    wsh WScriptHash |
    swInvalid WitnessProgram |
    tr XonlyPubkey |
    trFuture WitnessProgram |
    swFuture (FutureSegwitVer, WitnessProgram)

--
-- Queries about specific transaction
-- (signature Txid | Outpoint -> _)
--

@internal
fn txStatus :: Txid -> TxStatus

@internal
fn txAmount :: Txid -> Sats

@internal
fn txLockTime :: Txid -> LockTime

@internal
fn txFee :: Txid -> Sats

@internal -- implementation provided by RGB core
fn txScriptPubkey :: Outpoint -> ScriptPubkey

@internal
fn txotAmount :: Outpoint -> Sats

--
-- Queries about blockchain state
--

@internal
fn heightTimestamp :: Height -> Timestamp

@internal
fn heightDifficulty :: Height -> Difficulty

@internal
fn currentHeight -> Height
```

## Compatibility


## Rationale


## Reference implementation


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
