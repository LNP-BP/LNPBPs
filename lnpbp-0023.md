```
LNPBP: 0023
Aliases: RGB23
Vertical: Smart contracts
Title: RGB verifiable-unique history log for audits (RGB-23)
Authors: Dr Maxim Orlovsky <orlovsky@lnp-bp.org>
Comments-URI: <https://github.com/LNP-BP/LNPBPs/discussions>
Status: Proposal
Type: Standards Track
Created: 2020-09-10
Updated: 2023-07-24
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
-- Defined by LNPBP-31 standard in `RGBContract.sty` file
import urn:ubideco:stl:6vbr9ZrtsD9aBjo5qRQ36QEZPVucqvRRjKCPqE8yPeJr#choice-little-boxer as RGBContract

import RGB21

interface RGB23
    global id :: RGB21.ContractId
    global deed* :: Entry

    global created :: RGBContract.Timestamp

    owned deedRight

    op Log :: deedRight, deed -> deedRight

data Entry ::
    type MimeType,
    data [Byte],
    resources { ResourceId -> Resource }

data ResourceId :: U16

data Resource ::
    type MimeType,
    digest Digest

-- TODO: add more digest algorithms
-- TODO: move digest to Contractum standard library to use in RGB21
data Digest :: sha256([U8 ^ 32]) | blake3([U8 ^ 32])
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
