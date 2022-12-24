```
LNPBP: 0024
Aliases: RGB24
Vertical: Smart contracts
Title: RGB interface for decentralized global name system (RGB-24)
Authors: Dr Maxim Orlovsky <orlovsky@lnp-bp.org>
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


## Design


## Specification

Interface specification is the following Contractum code:

```haskell
data Ident :: [Alphanumeric+]

data Resolve :: ... -- TODO: Define data which is a name resolution

interface RGB24
    global Root? :: RGB24.ContractId
    global Name :: Ident
    global Registry{*} :: (Ident, Resolve)

    owned Registar

    op register :: Registar -> Registar <- Registry+
                !! noRoot |
                   invalidRoot |
                   incompleteRegistry
```

## Compatibility


## Rationale


## Reference implementation

```haskell
schema BaseRegistry implements RGB24
    op register :: Registar -> Registar <- entries Registry+
                !! noRoot |
                   unregisteredName |
                   incompleteRoot |
                   incompleteRegistry
        let root := self.Root !! noRoot
        let parent = root.Registry
        parent.isFullyKnown !! incompleteRoot
        parent.Registry.has self.Name !! unregisteredName
        let registry := self.Registry.isFullyKnown !! incompleteRegistry
        entries =|> self.Registry.has !! repeatedEntry
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