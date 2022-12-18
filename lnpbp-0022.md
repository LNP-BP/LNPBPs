```
LNPBP: 0022
Vertical: Smart contracts
Title: RGB reputation and identity interface (RGB-22)
Authors: Dr Maxim Orlovsky <orlovsky@lnp-bp.org>,
         Sabina Sachtachtinskagia
Comments-URI: <https://github.com/LNP-BP/LNPBPs/discussions>
Status: Proposal
Type: Standards Track
Created: 2020-09-10
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
    fact [Fact],
    attestedBy ContractId,
    signature Sig

interface RGB22 :: Identity
    global Name :: Unicode
    global Emails :: {Email}
    global Facts :: {Attestation}
    global Photo :: (MimeType, [Byte+])?

    owned NameRight
    owned AttestRight
    owned RevokableKey+ :: Pubkey

    op Revoke :: RevokableKey -> RevokableKey?
    op Rename :: NameRight -> NameRight <- Name, Emails
    op Attest :: AttestRight -> AttestRight, {RevokableKey} <- {Attestation}
```

## Compatibility


## Rationale


## Reference implementation

```haskell
use Secp256k1, Ed25519
import RGB21

fn check_key :: key Pubkey -> res Bool
    res = match key
    | secp256k1(k) => Secp256k1.check_key k
    | curve25519(k) => Ed25519.check_key k

schema BaseIdentity
    global Name :: Unicode
    global Emails :: {Email}
    global Facts :: {Attestation}
    global Photo :: (MimeType, [Byte+])?

    owned NameRight
    owned AttestRight
    owned RevokableKey :: Pubkey

    genesis :: Name, Emails, keys {RevokableKey+}
        keys => key -> assert! check_key key
     
    op Revoke :: old RevokableKey -> new RevokableKey?
        assert! check_key new
        assert! old != new
        
    op Rename :: NameRight -> NameRight <- Name, Emails
    
    op Attest :: AttestRight -> AttestRight, {RevokableKey} <- atts {Attestation}
        atts => a -> match a.signature
        | secp256k1(key, sig) => assert! Secp256k1.verify key, sig
        | ed25519k1(key, sig) => assert! Ed25519.verify key, sig

implement RGB21 for BaseIdentity
```


## Acknowledgements


## References


## Test vectors

```haskell
use BaseIdentity from identity.con

contract meSatoshiNakamoto implements BaseIdentity
    assign name Name := "Satoshi Nakamoto"
    assign email Emails := {"satoshi@nakam.oto"}
    assign keys Keys := [
        (seal: fac503c4641c3deda72a2d00bc9d6ff1094b15276c386efea403746a91436772:1,
         state: Pubkey.secp256k1(0x028730eeeec41802621d177507b086f390ae600ba3ca5e428b13913af4c2cd25b3))
    ]

transition iLostMyKey executes Revoke
    close meSatoshiNakamoto.keys[0].seal
    assign RevocableKey := (seal: ~:1, state: Pubkey.curve25519(0x0219db0a4e0eb8cb833608c08d76b9b279ec44a851ab82cc6fd68a9b32624bfa8b));
```

```console
$ rgb-cli exec satoshi.con
Registering schema ... success
Issuing contract meSatoshiNakamoto ... success
Preparing transition iLostMyKey ... saved to `iLostMyKey.rgb` and `iLostMyKey.psbt`
```

```console
$ rgb-cli read rgb21 ______
name: Satoshi Nakamoto
emails:
    - satoshi@nakam.oto
facts: []
photo: ~
revokableKey:
    - seal: ___:1
      state:
        alt: curve25519
        val: 0x0219db0a4e0eb8cb833608c08d76b9b279ec44a851ab82cc6fd68a9b32624bfa8b
```


## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.
