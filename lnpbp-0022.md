```
LNPBP: 0022
Aliases: RGB22
Vertical: Smart contracts
Title: RGB reputation and identity interface (RGB-22)
Authors: Dr Maxim Orlovsky <orlovsky@lnp-bp.org>,
         Sabina Sachtachtinskagia
Comments-URI: <https://github.com/LNP-BP/LNPBPs/discussions>
Status: Draft
Type: Standards Track
Created: 2020-09-10
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

Single-use-seal primitive represents an efficient enhancement for WoT model of
key revocation: it provides independent parties an ability to detect and 
timestamp key revocation without contacting the owner of the original identity.

RGB-22 standard is created to leverage this advantage and provide an 
alternative WoT solution to GPG/PGP. It can be combined with Storm protocol,
as a better alternative to PGP key servers.


## Design


## Specification

Interface specification is the following Contractum code:

```haskell
data XonlyPubkey :: [Byte ^ 32]
data SchnorrSig :: [Byte ^ 64]
data EdKey :: [Byte ^ 32]
data EdSig :: [Byte ^ 64)
data Fact :: [Byte+] -- JSON encoded string

data Pubkey :: secp256k1(XonlyPubkey) | curve25519(EdKey)
data Sig ::
    secp256k1(XonlyPubkey, SchnorrSig) | 
    ed25519(EdKey, EdSig)

data Attestation ::
    facts [Fact+],
    attestedBy RGB22.ContractId,
    signature Sig

interface RGB22
    global Name :: [Unicode+]

    global Emails{+} :: Email
    global Facts{*} :: Attestation
    global Photo? :: (MimeType, [Byte])

    owned NameRight
    owned AttestRight
    owned RevokableKey{+} :: Pubkey

    op revoke :: RevokableKey -> RevokableKey?
    op rename :: NameRight -> NameRight <- Name, Emails, Photo?
    op attest :: AttestRight -> AttestRight, {RevokableKey} <- {Attestation}
```

## Compatibility


## Rationale


## Reference implementation

```haskell
use Secp256k1, Ed25519
import RGB22

fn checkKey :: key Pubkey !! invalidKey
    key:
        Pubkey.secp256k1(k) -> Secp256k1.checkKey k !! invalidKey
        Pubkey.curve25519(k) -> Ed25519.checkKey k  !! invalidKey

schema BaseIdentity
    global Name :: [Unicode+]
    global Emails{+} :: Email
    global Facts{*} :: Attestation
    global Photo? :: (MimeType, [Byte])

    owned NameRight
    owned AttestRight
    owned RevokableKey{+} :: Pubkey

    genesis :: name Name, emails {Emails+}, keys {RevokableKey+} 
            !! invalidKey
        keys => key -> checkKey key
     
    op revoke :: old RevokableKey -> new RevokableKey?
              !! invalidKey
               | sameKey
        checkKey new
        !(old =? new) !! sameKey

    op rename :: NameRight -> NameRight <- Name, {Emails+}

    op attest :: AttestRight -> AttestRight, {RevokableKey} 
              <- atts {Attestation}
              !! invalidSig
        atts => a -> a.signature:
            Sig.secp256k1(key, sig) => Secp256k1.verify key, sig !! invalidSig
            Sig.ed25519k1(key, sig) => Ed25519.verify key, sig !! invalidSig

implement RGB21 for BaseIdentity
```


## Acknowledgements


## References


## Test vectors

```haskell
use BaseIdentity from identity.con

let orig = Seal(fac503c4641c3deda72a2d00bc9d6ff1094b15276c386efea403746a91436772, 1)

contract meSatoshiNakamoto := BaseIdentity (
    name := "Satoshi Nakamoto"
    emails := {"satoshi@nakam.oto"}
    keys := [
        (orig, Pubkey.secp256k1(0x028730eeeec41802621d177507b086f390ae600ba3ca5e428b13913af4c2cd25b3))
    ]
)
transition iLostMyKey := BaseIdentity.revoke (
    old := orig,
    new := (Seal(~:1), Pubkey.curve25519(0x0219db0a4e0eb8cb833608c08d76b9b279ec44a851ab82cc6fd68a9b32624bfa8b))
)
```

```console
$ rgb-cli exec satoshi.con
Registering schema ... success
Issuing contract meSatoshiNakamoto ... success
Preparing transition iLostMyKey ... saved to `iLostMyKey.rgb` and `iLostMyKey.psbt`
```

```console
$ rgb-cli state -i rgb22 rgb1______
name: Satoshi Nakamoto
emails:
  - satoshi@nakam.oto
facts: []
photo: ~
revokableKey:
  - 4459eaee3f84a1cd7529534d99b553a633671582c42640438071930e741253cf:1: 
    curve25519
      0: 0219db0a4e0eb8cb833608c08d76b9b279ec44a851ab82cc6fd68a9b32624bfa8b
```


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