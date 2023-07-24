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
Updated: 2023-07-24
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
-- Defined by LNPBP-31 standard in `RGBContract.sty` file
import urn:ubideco:stl:6vbr9ZrtsD9aBjo5qRQ36QEZPVucqvRRjKCPqE8yPeJr#choice-little-boxer as RGBContract

import urn:ubideco:stl:9KALDYR8Nyjq4FdMW6kYoL7vdkWnqPqNuFnmE9qHpNjZ#justice-rocket-type as Std

interface RGB24
    global root :: RGBContract.ContractId?
    global name :: Std.Ident
    global {registry} :: Record

    global created :: RGBContract.Timestamp

    owned registar

    op Register :: registar, {registry ^ 1..0xFFFF} -> registar
                !! noRoot |
                   invalidRoot |
                   incompleteRegistry

data Hostname :: [Std.AlphaNumDash ^ 1..63]
data DomainName :: [Hostname ^ 1..0xFF]

data Record :: host Hostname, entry Entry
data Entry :: a IPv4 | aaaa IPv6 | cname DomainName | sub RGBContract.ContractId
```

## Compatibility


## Rationale


## Reference implementation

```haskell
schema BaseRegistry implements RGB24
    op Register :: registar, entries {registry+} -> registar
                !! RegistrationError
        self.root => root -> (
            let parent = (root.contract !! noRoot).registry
            parent.isFullyKnown !! incompleteRoot
            let myself = parent.registry.get self.Name !! unregisteredName
            myself: Resolve.subdomain(contract) =>
                contract =? self.contractId !! selfNotRegistered
        )
        let registry := self.registry.isFullyKnown !! incompleteRegistry
        entries =|> self.registry.has !! repeatedEntry
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
