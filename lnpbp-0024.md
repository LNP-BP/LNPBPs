```
LNPBP: 0024
Aliases: RGB24
Vertical: Smart contracts
Title: RGB decentralized global name system (RGB-24)
Authors: Dr Maxim Orlovsky <orlovsky@lnp-bp.org>
Comments-URI: <https://github.com/LNP-BP/LNPBPs/discussions>
Status: Draft
Type: Standards Track
Created: 2020-09-10
Updated: 2022-12-23
Finalized: ~
License: GPL-3.0
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
interface RGB24
    global Root :: Self.ContractId?
    global Name :: Ident
    global Registry { Ident } :: Resolve

    owned Registar

    op register :: Registar -> Registar <- {Registry+}
                !! noRoot |
                   invalidRoot |
                   incompleteRegistry

data Ident :: [Alphanumeric+]

data Resolve :: record Record | subdomain Subdomain
data Record :: a IPv4 | aaaa IPv6 | cname Ident -- ...
data Subdomain :: RGB24.ContractId
```

## Compatibility


## Rationale


## Reference implementation

```haskell
schema BaseRegistry implements RGB24
    op register :: Registar -> Registar <- entries {Registry+}
                !! RegistrationError
        self.Root => root -> (
            let parent = (root.contract !! noRoot).Registry
            parent.isFullyKnown !! incompleteRoot
            let myself = parent.Registry.get self.Name !! unregisteredName
            myself: Resolve.subdomain(contract) => 
                contract =? self.ContractId !! selfNotRegistered
        )
        let registry := self.Registry.isFullyKnown !! incompleteRegistry
        entries =|> self.Registry.has !! repeatedEntry
        -- the above is the same as
        -- entries => entry -> self.registry.has entry !! repeatedEntry
        
data RegistrationError :: 
    noRoot |
    unregisteredName |
    incompleteRoot |
    incompleteRegistry |
    selfNotRegistered
```


## Acknowledgements


## References


## Copyright

This document is licensed under the GPL-3.0 license.
