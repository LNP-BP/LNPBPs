```
LNPBP: 0035
Layer: OSI Application (i7)
Vertical: Lightning network protocol
Title: Channel funding with RGB assets
Authors: Dr Maxim Orlovsky <orlovsky@lnp-bp.org>
Comments-URI: https://github.com/LNP-BP/lnpbps
Status: Draft
Type: Standards Track
Created: 2021-03-13
Finalized: not yet
License: CC0-1.0
```

## Specification

Two steps:
1. Propose funding with asset
2. Get funding confirmation

### `propose_rgb_funding`

1. Channel id
2. Asset id
3. Consignment id
4. Fragment no
5. Total fragments
6. Initial (partial) consignment data
7. Disclosure with state transition and anchor for the commitment transaction
8. Signature for commitment transaction
9. If witness commitment goes into an HTLC output, signatures for descending
   HTLCs

Consignment must assign funds to the channel funding transaction output;
must be valid and properly revealed. Disclosure must have an transition fully
revealed, while the anchor may be kept fully concealed.

`propose_rgb_funding` may be followed by multiple Bifrost unencrypted
`consignment` messages, if the consignment does not fit a single message

### `ack_rgb_funding`

1. Channel id
2. Consignment id
8. Signature for commitment transaction
9. If witness commitment goes into an HTLC output, signatures for descending
   HTLCs

NB: The witness transaction assigning the funds must not be published by the
proposing party until `ack_rgb_funding` is received
