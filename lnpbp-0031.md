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
  - [Version 0.10](#version-010)
  - [Work in progress for the next version](#work-in-progress-for-the-next-version)
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

### Version 0.10

The specification is the actual Standard Contractum Library code:

`Types.sty`, semantic lib id `olivia_angel_micro_Fd3Lx2smvwdKhRCkhMs2du3CAYyKqJneNSf1g72WjBoq`:
```haskell
typelib StdLib -- version v1.1.1-2024.04.20.A

data Alpha            :: A:65 | B:66 | C:67 | D:68 | E:69 | F:70 | G:71 | H:72 | I:73 | J:74 | K:75 | L:76 | M:77 | N:78 | O:79 | P:80 | Q:81 | R:82 | S:83 | T:84 | U:85 | V:86 | W:87 | X:88 | Y:89 | Z:90 | a:97 | b:98 | c:99 | d:100 | e:101 | f:102 | g:103 | h:104 | i:105 | j:106 | k:107 | l:108 | m:109 | n:110 | o:111 | p:112 | q:113 | r:114 | s:115 | t:116 | u:117 | v:118 | w:119 | x:120 | y:121 | z:122
data AlphaCaps        :: A:65 | B:66 | C:67 | D:68 | E:69 | F:70 | G:71 | H:72 | I:73 | J:74 | K:75 | L:76 | M:77 | N:78 | O:79 | P:80 | Q:81 | R:82 | S:83 | T:84 | U:85 | V:86 | W:87 | X:88 | Y:89 | Z:90
data AlphaCapsNum     :: zero:48 | one:49 | two:50 | three:51 | four:52 | five:53 | six:54 | seven:55 | eight:56 | nine:57 | A:65 | B:66 | C:67 | D:68 | E:69 | F:70 | G:71 | H:72 | I:73 | J:74 | K:75 | L:76 | M:77 | N:78 | O:79 | P:80 | Q:81 | R:82 | S:83 | T:84 | U:85 | V:86 | W:87 | X:88 | Y:89 | Z:90
data AlphaNum         :: zero:48 | one:49 | two:50 | three:51 | four:52 | five:53 | six:54 | seven:55 | eight:56 | nine:57 | A:65 | B:66 | C:67 | D:68 | E:69 | F:70 | G:71 | H:72 | I:73 | J:74 | K:75 | L:76 | M:77 | N:78 | O:79 | P:80 | Q:81 | R:82 | S:83 | T:84 | U:85 | V:86 | W:87 | X:88 | Y:89 | Z:90 | a:97 | b:98 | c:99 | d:100 | e:101 | f:102 | g:103 | h:104 | i:105 | j:106 | k:107 | l:108 | m:109 | n:110 | o:111 | p:112 | q:113 | r:114 | s:115 | t:116 | u:117 | v:118 | w:119 | x:120 | y:121 | z:122
data AlphaNumLodash   :: zero:48 | one:49 | two:50 | three:51 | four:52 | five:53 | six:54 | seven:55 | eight:56 | nine:57 | A:65 | B:66 | C:67 | D:68 | E:69 | F:70 | G:71 | H:72 | I:73 | J:74 | K:75 | L:76 | M:77 | N:78 | O:79 | P:80 | Q:81 | R:82 | S:83 | T:84 | U:85 | V:86 | W:87 | X:88 | Y:89 | Z:90 | lodash:95 | a:97 | b:98 | c:99 | d:100 | e:101 | f:102 | g:103 | h:104 | i:105 | j:106 | k:107 | l:108 | m:109 | n:110 | o:111 | p:112 | q:113 | r:114 | s:115 | t:116 | u:117 | v:118 | w:119 | x:120 | y:121 | z:122
data AlphaSmall       :: a:97 | b:98 | c:99 | d:100 | e:101 | f:102 | g:103 | h:104 | i:105 | j:106 | k:107 | l:108 | m:109 | n:110 | o:111 | p:112 | q:113 | r:114 | s:115 | t:116 | u:117 | v:118 | w:119 | x:120 | y:121 | z:122
data AsciiPrintable   :: space:32 | excl:33 | quotes:34 | hash:35 | dollar:36 | percent:37 | ampersand:38 | apostrophe:39 | bracketL:40 | bracketR:41 | asterisk:42 | plus:43 | comma:44 | minus:45 | dot:46 | slash:47 | zero:48 | one:49 | two:50 | three:51 | four:52 | five:53 | six:54 | seven:55 | eight:56 | nine:57 | colon:58 | semiColon:59 | less:60 | equal:61 | greater:62 | question:63 | at:64 | A:65 | B:66 | C:67 | D:68 | E:69 | F:70 | G:71 | H:72 | I:73 | J:74 | K:75 | L:76 | M:77 | N:78 | O:79 | P:80 | Q:81 | R:82 | S:83 | T:84 | U:85 | V:86 | W:87 | X:88 | Y:89 | Z:90 | sqBracketL:91 | backSlash:92 | sqBracketR:93 | caret:94 | lodash:95 | backtick:96 | a:97 | b:98 | c:99 | d:100 | e:101 | f:102 | g:103 | h:104 | i:105 | j:106 | k:107 | l:108 | m:109 | n:110 | o:111 | p:112 | q:113 | r:114 | s:115 | t:116 | u:117 | v:118 | w:119 | x:120 | y:121 | z:122 | cBracketL:123 | pipe:124 | cBracketR:125 | tilde:126
data Bool             :: false:0 | true:1
data Dec              :: zero:48 | one:49 | two:50 | three:51 | four:52 | five:53 | six:54 | seven:55 | eight:56 | nine:57
data HexDecCaps       :: zero:48 | one:49 | two:50 | three:51 | four:52 | five:53 | six:54 | seven:55 | eight:56 | nine:57 | ten:65 | eleven:66 | twelve:67 | thirteen:68 | fourteen:69 | fifteen:70
data HexDecSmall      :: zero:48 | one:49 | two:50 | three:51 | four:52 | five:53 | six:54 | seven:55 | eight:56 | nine:57 | ten:97 | eleven:98 | twelve:99 | thirteen:100 | fourteen:101 | fifteen:102
data U4               :: v0:0 | v1:1 | v2:2 | v3:3 | v4:4 | v5:5 | v6:6 | v7:7 | v8:8 | v9:9 | v10:10 | v11:11 | v12:12 | v13:13 | v14:14 | v15:15
```

`Bitcoin.sty`, semantic lib id `panel_chamber_ohio_GWzzfwBqzA5BUVER6hqq3rBdDs6UUJ15w1T8ys6WrNr5`:
```haskell
typelib Bitcoin -- version v0.10.1-2023.04.20.B

data LockTime         :: U32
data Outpoint         :: txid Txid, vout Vout
data Sats             :: U64
data ScriptBytes      :: [U8 ^ ..0xffffffffffffffff]
data ScriptPubkey     :: ScriptBytes
data SeqNo            :: U32
data SigScript        :: ScriptBytes
data Tx               :: version TxVer
                       , inputs [TxIn ^ ..0xffffffffffffffff]
                       , outputs [TxOut ^ ..0xffffffffffffffff]
                       , lockTime LockTime
data TxIn             :: prevOutput Outpoint
                       , sigScript SigScript
                       , sequence SeqNo
                       , witness Witness
data TxOut            :: value Sats, scriptPubkey ScriptPubkey
data TxVer            :: U32
data Txid             :: [U8 ^ 32]
data Vout             :: U32
data Witness          :: [[U8 ^ ..0xffffffffffffffff] ^ ..0xffffffffffffffff]
```

`CommitVerify.sty`, semantic lib id `texas_year_ethnic_CPr8tcdPqWZ3KP8dXNPYavTEkbn8PG7CoJHtfwDFKRHJ`
```haskell
typelib CommitVerify -- version v0.10.0-2023.03.05.A

data Message          :: [U8 ^ 32]
data ProtocolId       :: [U8 ^ 32]
data MerkleNode       :: [U8 ^ 32]

data TreeNode         :: concealedNode (depth StdLib.U4, hash MerkleNode)
                       | commitmentLeaf (protocolId ProtocolId, message Message)

data MerkleTree       :: depth StdLib.U4
                       , entropy U64
                       , messages {[Byte ^ 32] -> Message}
                       , map {U16 -> ProtocolId, Message}

data MerkleBlock      :: depth StdLib.U4
                       , crossSection [TreeNode]
                       , entropy U64?

data MerkleProof      :: pos U16, path [MerkleNode]
```

`RGBContracts.sty`, semantic lib id ``:
```haskell
typelib RGBTypes -- version v0.10.1-2023.04.20.C

import panel_chamber_ohio_GWzzfwBqzA5BUVER6hqq3rBdDs6UUJ15w1T8ys6WrNr5 as Bitcoin
import olivia_angel_micro_Fd3Lx2smvwdKhRCkhMs2du3CAYyKqJneNSf1g72WjBoq as _

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

data Amount :: Zk64 -- fungible asset amount

data Ticker :: [AlphaCapsNum ^ 1..8]
data Name :: [AsciiPrintable ^ 1..40]
data Details :: [Unicode ^ 1..256]

data AssetNaming ::
    ticker Ticker,
    name Name,
    details Details?

data DivisibleAssetSpec ::
    naming AssetNaming, 
    precision Precision
    
data RicardianContract :: [Unicode]

-- `reg-name` defined by MIME spec
-- TODO: filter on the character level with a more precise metric
data MediaRegName :: [AsciiPrintable ^ 1..127]

data MediaType ::
    type MediaRegName,
    subtype MediaRegName,
    -- We do not support other parameters
    charset MediaRegName

-- proof of reserves
data PoR ::
    utxo Bitcoin.Outpoint,
    proof [Bytes] -- auxilary data which are schema-specific
```

### Work in progress for the next version

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

data Height :: U32
data Timestamp :: I64
data Difficulty: U32

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
