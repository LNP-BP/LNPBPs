```
LNPBP: 0053
Layer: OSI Application (i7)
Vertical: Lightning network protocol
Title: Muli-peer payment channels for Bifrost
Author: Dr Maxim Orlovsky <orlovsky@lnp-bp.org>
Comments-URI: https://github.com/LNP-BP/lnpbps/pulls/97
Status: Draft
Type: Standards Track
Created: 2021-11-14
Finalized: not yet
License: CC0-1.0
```

## Abstract

## Background

## Motivation

## Design

## Specification

### Messaging


### Funding transaction

Transaction containing multiple inputs, each of which MUST signal S2C commitment type.

Output descriptor: `tr(musig(KEY_LOCAL_FUNDING, KEY_REMOTE_FUNDING))`


### Commitment transaction

Input:
- Spends funding output
- nSeq: `0x80800000 + COMMITMENT_NO & 0xFFFF` (signals S2C DBC)
- Witness stack:
  - `KEY_LOCAL_FUNDING + KEY_REMOTE_FUNDING`
  - `SIG*(KEY_LOCAL_FUNDING + KEY_REMOTE_FUNDING)`
  where * denotes optional presence of sign-to-contract commitment

Outputs:
- "To local": `tr(KEY_REVOCATION, and_v(v:pk(KEY_LOCAL),older(SELF_DELAY)))`
  * `KEY_LOCAL`: proposed by the local node
  * `SELF_DELAY`: proposed by the remote node, may be rejected by local
- "To remote": `tr(musig(KEY_LOCAL, KEY_REMOTE), and_v(v:pk(KEY_REMOTE),older(REMOTE_DELAY))`
  * `KEY_LOCAL`: proposed by the local node
  * `KEY_REMOTE`: propsoed by the remote node
  * `REMOTE_DELAY`: proposed by the local node, may be rejected by remote
- "Anchor": `tr(musig(KEY_LOCAL, KEY_REMOTE), {v:pk(KEY_LOCAL), v:pk(KEY_REMOTE)})`
  * `KEY_LOCAL`: proposed by the local node
  * `KEY_REMOTE`: propsoed by the remote node
  * The ooutput must contain a fixed amount of satoshis above dust limit proposed by
    the node adding this output to the channel proposal


## Compatibility

## Rationale

## Reference implementation

## Acknowledgements

## References

## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.

## Test vectors
