```
LNPBP: 0031
Vertical: Smart contracts
Title: Standard Contractum Libraries (SCL)
Authors: Dr Maxim Orlovsky <orlovsky@lnp-bp.org>
Comments-URI: <https://github.com/LNP-BP/LNPBPs/discussions/141>
Status: Proposal
Type: Standards Track
Created: 2022-12-23
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

`Std.sty`, semantic lib id `urn:ubideco:stl:9KALDYR8Nyjq4FdMW6kYoL7vdkWnqPqNuFnmE9qHpNjZ#justice-rocket-type`:
```haskell
typelib Std

-- no dependencies

-- urn:ubideco:semid:55LGxTk42rZVmuXepF2FJNfP52h3QG5pBnBaY1VnnvnP#quiet-opinion-saddle
data Alpha            :: A:65 | B:66 | C:67 | D:68
                       | E:69 | F:70 | G:71 | H:72
                       | I:73 | J:74 | K:75 | L:76
                       | M:77 | N:78 | O:79 | P:80
                       | Q:81 | R:82 | S:83 | T:84
                       | U:85 | V:86 | W:87 | X:88
                       | Y:89 | Z:90 | a:97 | b:98
                       | c:99 | d:100 | e:101 | f:102
                       | g:103 | h:104 | i:105 | j:106
                       | k:107 | l:108 | m:109 | n:110
                       | o:111 | p:112 | q:113 | r:114
                       | s:115 | t:116 | u:117 | v:118
                       | w:119 | x:120 | y:121 | z:122

-- urn:ubideco:semid:43EA5YjDDUMdgMApG3xuUWGSzDXKr6U5b9gtgLwuqCc3#simon-pegasus-stop
data AlphaCaps        :: A:65 | B:66 | C:67 | D:68
                       | E:69 | F:70 | G:71 | H:72
                       | I:73 | J:74 | K:75 | L:76
                       | M:77 | N:78 | O:79 | P:80
                       | Q:81 | R:82 | S:83 | T:84
                       | U:85 | V:86 | W:87 | X:88
                       | Y:89 | Z:90

-- urn:ubideco:semid:7U5NvJNf343ZzFXsqW2DBYtTSvrb3YdL6oxYd2BaMsVr#magnet-section-latin
data AlphaCapsNum     :: zero:48 | one:49 | two:50 | three:51
                       | four:52 | five:53 | six:54 | seven:55
                       | eight:56 | nine:57 | A:65 | B:66
                       | C:67 | D:68 | E:69 | F:70
                       | G:71 | H:72 | I:73 | J:74
                       | K:75 | L:76 | M:77 | N:78
                       | O:79 | P:80 | Q:81 | R:82
                       | S:83 | T:84 | U:85 | V:86
                       | W:87 | X:88 | Y:89 | Z:90

-- urn:ubideco:semid:DZX8CtQMz2kGByNeWpSpWzor5EPoeQ1LRsSWcR13w9bH#disco-ibiza-mile
data AlphaNum         :: zero:48 | one:49 | two:50 | three:51
                       | four:52 | five:53 | six:54 | seven:55
                       | eight:56 | nine:57 | A:65 | B:66
                       | C:67 | D:68 | E:69 | F:70
                       | G:71 | H:72 | I:73 | J:74
                       | K:75 | L:76 | M:77 | N:78
                       | O:79 | P:80 | Q:81 | R:82
                       | S:83 | T:84 | U:85 | V:86
                       | W:87 | X:88 | Y:89 | Z:90
                       | a:97 | b:98 | c:99 | d:100
                       | e:101 | f:102 | g:103 | h:104
                       | i:105 | j:106 | k:107 | l:108
                       | m:109 | n:110 | o:111 | p:112
                       | q:113 | r:114 | s:115 | t:116
                       | u:117 | v:118 | w:119 | x:120
                       | y:121 | z:122

-- urn:ubideco:semid:4UQSpEBq39vFqEBYDaLEMaQ6qJYGDiEFsGFYzv7U7ipA#good-trumpet-today
data AlphaNumDash     :: dash:45 | zero:48 | one:49 | two:50
                       | three:51 | four:52 | five:53 | six:54
                       | seven:55 | eight:56 | nine:57 | A:65
                       | B:66 | C:67 | D:68 | E:69
                       | F:70 | G:71 | H:72 | I:73
                       | J:74 | K:75 | L:76 | M:77
                       | N:78 | O:79 | P:80 | Q:81
                       | R:82 | S:83 | T:84 | U:85
                       | V:86 | W:87 | X:88 | Y:89
                       | Z:90 | a:97 | b:98 | c:99
                       | d:100 | e:101 | f:102 | g:103
                       | h:104 | i:105 | j:106 | k:107
                       | l:108 | m:109 | n:110 | o:111
                       | p:112 | q:113 | r:114 | s:115
                       | t:116 | u:117 | v:118 | w:119
                       | x:120 | y:121 | z:122

-- urn:ubideco:semid:8iBe2dh8beD1KUairdqCacEcxAr4h55XfUQN2PspWXjz#north-sound-salsa
data AlphaNumLodash   :: zero:48 | one:49 | two:50 | three:51
                       | four:52 | five:53 | six:54 | seven:55
                       | eight:56 | nine:57 | A:65 | B:66
                       | C:67 | D:68 | E:69 | F:70
                       | G:71 | H:72 | I:73 | J:74
                       | K:75 | L:76 | M:77 | N:78
                       | O:79 | P:80 | Q:81 | R:82
                       | S:83 | T:84 | U:85 | V:86
                       | W:87 | X:88 | Y:89 | Z:90
                       | lodash:95 | a:97 | b:98 | c:99
                       | d:100 | e:101 | f:102 | g:103
                       | h:104 | i:105 | j:106 | k:107
                       | l:108 | m:109 | n:110 | o:111
                       | p:112 | q:113 | r:114 | s:115
                       | t:116 | u:117 | v:118 | w:119
                       | x:120 | y:121 | z:122

-- urn:ubideco:semid:HmLtNhtTNv8cdSDzKcU3p1i3jcJS6TWkrRCw1vYABFJG#song-accent-mammal
data AlphaSmall       :: a:97 | b:98 | c:99 | d:100
                       | e:101 | f:102 | g:103 | h:104
                       | i:105 | j:106 | k:107 | l:108
                       | m:109 | n:110 | o:111 | p:112
                       | q:113 | r:114 | s:115 | t:116
                       | u:117 | v:118 | w:119 | x:120
                       | y:121 | z:122

-- urn:ubideco:semid:2NFrhqQqGNDA4HujyTW2pmcjtrN5sbtFfpPFXPPYcGER#aloha-lunar-felix
data Ascii            :: nul:0 | soh:1 | stx:2 | etx:3
                       | eot:4 | enq:5 | ack:6 | bel:7
                       | bs:8 | ht:9 | lf:10 | vt:11
                       | ff:12 | cr:13 | so:14 | si:15
                       | dle:16 | dc1:17 | dc2:18 | dc3:19
                       | dc4:20 | nack:21 | syn:22 | etb:23
                       | can:24 | em:25 | sub:26 | esc:27
                       | fs:28 | gs:29 | rs:30 | us:31
                       | space:32 | excl:33 | quotes:34 | hash:35
                       | dollar:36 | percent:37 | ampersand:38 | apostrophe:39
                       | bracketL:40 | bracketR:41 | asterisk:42 | plus:43
                       | comma:44 | minus:45 | dot:46 | slash:47
                       | zero:48 | one:49 | two:50 | three:51
                       | four:52 | five:53 | six:54 | seven:55
                       | eight:56 | nine:57 | colon:58 | semiColon:59
                       | less:60 | equal:61 | greater:62 | question:63
                       | at:64 | A:65 | B:66 | C:67
                       | D:68 | E:69 | F:70 | G:71
                       | H:72 | I:73 | J:74 | K:75
                       | L:76 | M:77 | N:78 | O:79
                       | P:80 | Q:81 | R:82 | S:83
                       | T:84 | U:85 | V:86 | W:87
                       | X:88 | Y:89 | Z:90 | sqBracketL:91
                       | backSlash:92 | sqBracketR:93 | caret:94 | lodash:95
                       | backtick:96 | a:97 | b:98 | c:99
                       | d:100 | e:101 | f:102 | g:103
                       | h:104 | i:105 | j:106 | k:107
                       | l:108 | m:109 | n:110 | o:111
                       | p:112 | q:113 | r:114 | s:115
                       | t:116 | u:117 | v:118 | w:119
                       | x:120 | y:121 | z:122 | cBracketL:123
                       | pipe:124 | cBracketR:125 | tilde:126 | del:127

-- urn:ubideco:semid:mbH4meZSjxky12xHm9pg3rw8VoGxEa6rXtt6dAMZLbt#diet-oxford-window
data AsciiPrintable   :: space:32 | excl:33 | quotes:34 | hash:35
                       | dollar:36 | percent:37 | ampersand:38 | apostrophe:39
                       | bracketL:40 | bracketR:41 | asterisk:42 | plus:43
                       | comma:44 | minus:45 | dot:46 | slash:47
                       | zero:48 | one:49 | two:50 | three:51
                       | four:52 | five:53 | six:54 | seven:55
                       | eight:56 | nine:57 | colon:58 | semiColon:59
                       | less:60 | equal:61 | greater:62 | question:63
                       | at:64 | A:65 | B:66 | C:67
                       | D:68 | E:69 | F:70 | G:71
                       | H:72 | I:73 | J:74 | K:75
                       | L:76 | M:77 | N:78 | O:79
                       | P:80 | Q:81 | R:82 | S:83
                       | T:84 | U:85 | V:86 | W:87
                       | X:88 | Y:89 | Z:90 | sqBracketL:91
                       | backSlash:92 | sqBracketR:93 | caret:94 | lodash:95
                       | backtick:96 | a:97 | b:98 | c:99
                       | d:100 | e:101 | f:102 | g:103
                       | h:104 | i:105 | j:106 | k:107
                       | l:108 | m:109 | n:110 | o:111
                       | p:112 | q:113 | r:114 | s:115
                       | t:116 | u:117 | v:118 | w:119
                       | x:120 | y:121 | z:122 | cBracketL:123
                       | pipe:124 | cBracketR:125 | tilde:126

-- urn:ubideco:semid:7ZhBHGSJm9ixmm8Z9vCX7i5Ga7j5xrW8t11nsb1Cgpnx#laser-madam-maxwell
data Bool             :: false:0 | true:1

-- urn:ubideco:semid:DfVXYs8NyS6G5QLTQMUELHWGkSoenXDw3ZFrHzG3LjMW#amanda-spider-diamond
data Dec              :: zero:48 | one:49 | two:50 | three:51
                       | four:52 | five:53 | six:54 | seven:55
                       | eight:56 | nine:57

-- urn:ubideco:semid:H5T3iaCVzmGH5BcotfpzcRNb5Z1ri27rwVrRhJ6UosU6#invest-moral-anvil
data HexDecCaps       :: zero:48 | one:49 | two:50 | three:51
                       | four:52 | five:53 | six:54 | seven:55
                       | eight:56 | nine:57 | ten:65 | eleven:66
                       | twelve:67 | thirteen:68 | fourteen:69 | fifteen:70

-- urn:ubideco:semid:CBhZBmVRHY5sgou91KEAQrxun6kQQdbCPRekEEoB5Lik#forum-sahara-email
data HexDecSmall      :: zero:48 | one:49 | two:50 | three:51
                       | four:52 | five:53 | six:54 | seven:55
                       | eight:56 | nine:57 | ten:97 | eleven:98
                       | twelve:99 | thirteen:100 | fourteen:101 | fifteen:102

-- urn:ubideco:semid:BrEhDdRPrktqBgsNbgsmUagRz9b5n5csfbmif8Y7Bcc8#east-invest-harvest
data U4               :: u4_0:0 | u4_1:1 | u4_2:2 | u4_3:3
                       | u4_4:4 | u4_5:5 | u4_6:6 | u4_7:7
                       | u4_8:8 | u4_9:9 | u4_10:10 | u4_11:11
                       | u4_12:12 | u4_13:13 | u4_14:14 | u4_15:15

-- urn:ubideco:semid:3MDHMYsJt8d1gUiyx5vGCWcNLQ7biek6UTjHg3ksW4Bf#ground-volume-singer
data U5               :: u5_0:0 | u5_1:1 | u5_2:2 | u5_3:3
                       | u5_4:4 | u5_5:5 | u5_6:6 | u5_7:7
                       | u5_8:8 | u5_9:9 | u5_10:10 | u5_11:11
                       | u5_12:12 | u5_13:13 | u5_14:14 | u5_15:15
                       | u5_16:16 | u5_17:17 | u5_18:18 | u5_19:19
                       | u5_20:20 | u5_21:21 | u5_22:22 | u5_23:23
                       | u5_24:24 | u5_25:25 | u5_26:26 | u5_27:27
                       | u5_28:28 | u5_29:29 | u5_30:30 | u5_31:31
```

`Bitcoin.sty`, semantic lib id `urn:ubideco:stl:6GgF7biXPVNcus2FfQj2pQuRzau11rXApMQLfCZhojgi#money-pardon-parody`:
```haskell
typelib Bitcoin

-- no dependencies

-- urn:ubideco:semid:4dDWWU4afiPN3q4AgCMuFRFhL4UDta2u5SrqrBzPvjby#tokyo-inch-program
data LockTime         :: U32

-- urn:ubideco:semid:FWt2MSo8A4nsYgYbuBqMRNLiKgtzvLBgUn774iKzTcuf#pocket-pegasus-frank
data Outpoint         :: txid Txid, vout Vout

-- urn:ubideco:semid:BEBz6h7AGjYSDRCxVHnjYkkkxzBsjN3EvyNiD4ZrzmRL#pyramid-spray-star
data Sats             :: U64

-- urn:ubideco:semid:3Y4AgjkFbDusgo3YqRDWv9BznDeAJEUDEPeEq1mpSkAR#maestro-source-jackson
data ScriptBytes      :: [Byte ^ ..0xffffffff]

-- urn:ubideco:semid:2ZAYpWKB2BQxeXXjpQDpYGZ7eXFM9qQxN9TcdTiQqeB8#bingo-maestro-silk
data ScriptPubkey     :: ScriptBytes

-- urn:ubideco:semid:5HtymNhYBhjqPkLLw9QVWZ62cLm57cZxgQTDUBBXtmL#rhino-time-rodent
data SeqNo            :: U32

-- urn:ubideco:semid:2gTMqAC393rBSGtBhDn8sJq3F3HtDosbqKDQTw9bHFyT#prelude-analyze-think
data SigScript        :: ScriptBytes

-- urn:ubideco:semid:DynChojW1sfr8VjSoZbmReHhZoU8u9KCiuwijgEGdToe#milk-gloria-prize
data Tx               :: version TxVer
                       , inputs [TxIn ^ ..0xffffffff]
                       , outputs [TxOut ^ ..0xffffffff]
                       , lockTime LockTime

-- urn:ubideco:semid:9Nf4Vvt3im8tFQSGzPWKfjfhsrkB8bf2XsLWfzywiFSv#antenna-crater-planet
data TxIn             :: prevOutput Outpoint
                       , sigScript SigScript
                       , sequence SeqNo
                       , witness Witness

-- urn:ubideco:semid:HutVbeKmYYrNun96Pi4T7YfYww7SeWxRFPZGDiZwoGZV#design-jacket-spirit
data TxOut            :: value Sats, scriptPubkey ScriptPubkey

-- urn:ubideco:semid:CLhr1zatQBSkCz9SiVrNoKB5igCZfF3hqRizfrviM6NR#english-natasha-virus
data TxVer            :: I32

-- urn:ubideco:semid:C1GfCrG7AXu2sFhRBspd7KpJK2YgyTkVy6pty5rZynRs#cowboy-diego-betty
data Txid             :: [Byte ^ 32]

-- urn:ubideco:semid:3HHRtSJW5fnGkdVW1EVDH7B97Y79WhwvKyyfsaBkuQkk#chrome-robin-gallop
data Vout             :: U32

-- urn:ubideco:semid:8mjN2CZj3Nhn2HjnKqTmEcN5vmyb3UJK8HSFW1uE3W2p#warning-saddle-period
data Witness          :: [[Byte ^ ..0xffffffff] ^ ..0xffffffff]
```

`CommitVerify.sty`, semantic lib id `urn:ubideco:stl:ZtHaBzu9ojbDahaGKEXe5v9DfSDxLERbLkEB23R6Q6V#rhino-cover-frog`
```haskell
typelib CommitVerify

import urn:ubideco:stl:9KALDYR8Nyjq4FdMW6kYoL7vdkWnqPqNuFnmE9qHpNjZ#justice-rocket-type as Std
-- Imports:
-- U5 := urn:ubideco:semid:3MDHMYsJt8d1gUiyx5vGCWcNLQ7biek6UTjHg3ksW4Bf#ground-volume-singer



-- urn:ubideco:semid:F8mU5NPc8Z5CMnkSFGdF5UxrPsdcBS6B5DCyP5kJPgWc#ventura-equal-think
data Commitment       :: [Byte ^ 32]

-- urn:ubideco:semid:qp6pMjMCcukxxZdkM2PtfNWfJjXKoVHXtXSBCsYjQwY#transit-bogart-nissan
data MerkleBlock      :: depth Std.U5 {- urn:ubideco:semid:3MDHMYsJt8d1gUiyx5vGCWcNLQ7biek6UTjHg3ksW4Bf#ground-volume-singer -}
                       , cofactor U16
                       , crossSection [TreeNode ^ ..0xffffffff]
                       , entropy U64?

-- urn:ubideco:semid:6kxYeCatpncbA9UiTdsFbxbxJdU56x6MdmTRkEeGAv6R#iceberg-rocket-velvet
data MerkleNode       :: [Byte ^ 32]

-- urn:ubideco:semid:9FbrjZLnMDfbrN9gEbWij5HNkxqAVaZBkoW2UvKdYw4B#canyon-exhibit-ravioli
data MerkleProof      :: pos U32
                       , cofactor U16
                       , path [MerkleNode ^ ..0x20]

-- urn:ubideco:semid:57jCv2LWrdn89GzuSYaH17f21N3su76uM2tEaG1dwwoT#russian-wedding-florida
data MerkleTree       :: depth Std.U5 {- urn:ubideco:semid:3MDHMYsJt8d1gUiyx5vGCWcNLQ7biek6UTjHg3ksW4Bf#ground-volume-singer -}
                       , entropy U64
                       , cofactor U16
                       , messages {ProtocolId -> ^ ..0xffffff Message}
                       , map {U32 -> ^ ..0xffffff ProtocolId, Message}

-- urn:ubideco:semid:4ajqScXjJ6wQ5af2zgBFzzP7k1qzD6DXXU28taQidCcA#shampoo-bishop-morgan
data Message          :: [Byte ^ 32]

-- urn:ubideco:semid:4GenVCt5Xq6xtnJDjT98FehgCS8rTmwEzbjwGkaUVjHz#gamma-banjo-corona
data ProtocolId       :: [Byte ^ 32]

-- urn:ubideco:semid:D7Q2eTnYyjN6gMZnZYrMG6gmRwmtnxyGLeqBbki8DFLv#greek-decimal-quiz
data TreeNode         :: concealedNode (depth Std.U5 {- urn:ubideco:semid:3MDHMYsJt8d1gUiyx5vGCWcNLQ7biek6UTjHg3ksW4Bf#ground-volume-singer -}, hash MerkleNode)
                       | commitmentLeaf (protocolId ProtocolId, message Message)

```

`RGBContract.sty`, semantic lib id `urn:ubideco:stl:6vbr9ZrtsD9aBjo5qRQ36QEZPVucqvRRjKCPqE8yPeJr#choice-little-boxer`:
```haskell
typelib RGBContract

import urn:ubideco:stl:6GgF7biXPVNcus2FfQj2pQuRzau11rXApMQLfCZhojgi#money-pardon-parody as Bitcoin
-- Imports:
-- Vout := urn:ubideco:semid:3HHRtSJW5fnGkdVW1EVDH7B97Y79WhwvKyyfsaBkuQkk#chrome-robin-gallop
-- Txid := urn:ubideco:semid:C1GfCrG7AXu2sFhRBspd7KpJK2YgyTkVy6pty5rZynRs#cowboy-diego-betty
-- Outpoint := urn:ubideco:semid:GeFZHi1RYCrrcH1LG4Fo2SWW5M6KLJ8yvoGkFjRWZaA9#dinner-yoga-danube

import urn:ubideco:stl:9KALDYR8Nyjq4FdMW6kYoL7vdkWnqPqNuFnmE9qHpNjZ#justice-rocket-type as Std
-- Imports:
-- AsciiPrintable := urn:ubideco:semid:mbH4meZSjxky12xHm9pg3rw8VoGxEa6rXtt6dAMZLbt#diet-oxford-window
-- AlphaCapsNum := urn:ubideco:semid:7U5NvJNf343ZzFXsqW2DBYtTSvrb3YdL6oxYd2BaMsVr#magnet-section-latin



-- urn:ubideco:semid:AC2a15L721Fw1YSudEvyX7vr8XjPVn4bPUrRhmZS4oJj#burma-picasso-granite
data Amount           :: U64

-- urn:ubideco:semid:Ep3efqbERhgbus3JbSaKn3Lm9gWtya9xoGYbAjoQhXaB#heavy-public-hostel
data AssetNaming      :: ticker Ticker
                       , name Name
                       , details Details?

-- urn:ubideco:semid:9t5kYLUwTpWjwh9eHB1NU3obZnj3qeTZzpZdcfYiqAV4#flame-unicorn-fruit
data Attachment       :: type MediaType, digest [Byte ^ 32]

-- urn:ubideco:semid:HtN246bWqDBKMgUJf7cKxERW9B2ostpVYPnAG2LVCKCX#gabriel-fiber-oregano
data BurnMeta         :: burnProofs {ProofOfReserves}

-- urn:ubideco:semid:tZLspSCzoPWcsyhL3Q9Tks45bGupxp9VRtvLzQfsBYS#symbol-medical-marion
data ContractData     :: terms RicardianContract, media Attachment?

-- urn:ubideco:semid:5Fb7RNdm2jWi7wndRaaU8Lwx76exafTvgQqt9owU9JwM#network-kayak-adam
data Details          :: [Unicode ^ 1..0xff]

-- urn:ubideco:semid:3p1E6oqjmmGPMHh6H4G3BrQU3iuwr7XRmmytiixPp1oh#elvis-alex-letter
data DivisibleAssetSpec :: naming AssetNaming, precision Precision

-- urn:ubideco:semid:5r9sYFUJy7Kd9FEZ1pe5v4BUPiy1Bg344pNPsENbb7X#alcohol-moral-needle
data IssueMeta        :: reserves {ProofOfReserves}

-- urn:ubideco:semid:Bn87eabCqLDccdn1qvtnaxtDrofnw1mBexxR8tSjkN7z#memphis-bicycle-roof
data MediaRegName     :: [MimeChar ^ 1..0x40]

-- urn:ubideco:semid:AUJnh2sR5dxk1TQRtXr7vYVTorbG4Tiy4LY14eQp9yV2#robert-decide-dispute
data MediaType        :: type MediaRegName
                       , subtype MediaRegName?
                       , charset MediaRegName?

-- urn:ubideco:semid:56Qs8Zfm2GAgewu9s7ffVb9xX6QiJhoDskxMhBoz723U#golf-antonio-courage
data MimeChar         :: excl:33 | hash:35 | dollar:36 | amp:38
                       | plus:43 | dash:45 | dot:46 | zero:48
                       | one:49 | two:50 | three:51 | four:52
                       | five:53 | six:54 | seven:55 | eight:56
                       | nine:57 | caret:94 | lodash:95 | a:97
                       | b:98 | c:99 | d:100 | e:101
                       | f:102 | g:103 | h:104 | i:105
                       | j:106 | k:107 | l:108 | m:109
                       | n:110 | o:111 | p:112 | q:113
                       | r:114 | s:115 | t:116 | u:117
                       | v:118 | w:119 | x:120 | y:121
                       | z:122

-- urn:ubideco:semid:6PbMuf2YBk8Ff4J15AZ1MBW8XbcAUsprYiF7QjusVrz7#crystal-visitor-tribune
data Name             :: [Std.AsciiPrintable {- urn:ubideco:semid:mbH4meZSjxky12xHm9pg3rw8VoGxEa6rXtt6dAMZLbt#diet-oxford-window -} ^ 1..0x28]

-- urn:ubideco:semid:7G6FJPNejRtmGZP4NPXXHTTozzH4cwdrwrdkB3gziMa1#union-drum-public
data Precision        :: indivisible:0 | deci:1 | centi:2 | milli:3
                       | deciMilli:4 | centiMilli:5 | micro:6 | deciMicro:7
                       | centiMicro:8 | nano:9 | deciNano:10 | centiNano:11
                       | pico:12 | deciPico:13 | centiPico:14 | femto:15
                       | deciFemto:16 | centiFemto:17 | atto:18

-- urn:ubideco:semid:zrXMtzeLgFy1NQd46y3CNb549tnukiuMEGJvqxRkyDW#liquid-owner-london
data ProofOfReserves  :: utxo Bitcoin.Outpoint {- urn:ubideco:semid:GeFZHi1RYCrrcH1LG4Fo2SWW5M6KLJ8yvoGkFjRWZaA9#dinner-yoga-danube -}, proof [Byte]

-- urn:ubideco:semid:2fnqF5VfphtEoAWWEXwqyAZwny3YhkbB5TAh4VpA5JxQ#bundle-turbo-verona
data RicardianContract :: [Unicode]

-- urn:ubideco:semid:9dzjKz1d9KyoGnEg6WFSAKnWMaCpg1Lh7p66cmFXbH9e#mike-atlas-store
data Ticker           :: [Std.AlphaCapsNum {- urn:ubideco:semid:7U5NvJNf343ZzFXsqW2DBYtTSvrb3YdL6oxYd2BaMsVr#magnet-section-latin -} ^ 1..0x8]

-- urn:ubideco:semid:7eMrzgjRCf7EFcBBf6evAE75NTerkJ7tBdJAKqNfVGVs#suzuki-castle-saint
data Timestamp        :: I64
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
data Pubkey :: uncompressed FullPubkey | compressed CompressedPubkey
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
