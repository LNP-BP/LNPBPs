```
LNPBP: 0055
Layer: OSI Application (i7)
Vertical: Lightning network protocol
Title: HTLC channel synchronization in Bifrost
Author: Dr Maxim Orlovsky <orlovsky@lnp-bp.org>
Comments-URI: https://github.com/LNP-BP/lnpbps/pulls/97
Status: Draft
Type: Standards Track
Created: 2021-11-14
Finalized: not yet
License: CC0-1.0
```

## Abstract

This document specifies upgrade of BOLT-1, 3, 4 and 5-based routed payments in
legacy network and covers payment routing via HTLC-based channel synchronization
in Bifrost using Taproot, TapScript and miniscript-based bitcoin output 
descriptors.

## Background

HTLC (hash-time-locked contracts) are used for coordination of state update between 
multiple UTXO-based channels. This was the first mechanism proposed for payment
routing in the original lightning network.

## Motivation

Legacy lightning network uses pre-Taproot scripts and non-Schnorr signatures, which
reduces its privacy and requires more onchain space, especially in cases of non-
cooperative channel closings.

## Design

## Specification

### Messaging

HTLC-based payment coordination is performed by using onion message transport from
LNPBP-52. This specification defines the following message types embedded as onion
payloads:

`HTLC_ADD`

`HTLC_EXPIRE`

`HTLC_FULFILL`

`HTLC_MALFORMED`

It also re-uses `SIG_UPDATE` and `REVOKE` messages from the core Bifrost protocol
(LNPBP-50).

### Transactions

Each HTLC contract adds an output to the *commitment transaction*. The output 
depends on whether this is an offered HTLC or received one.

#### Offered HTLC outputs

<!-- single branch version: 
    ```
    tr(KEY_REVOCATION, and_v(
        and_v(
            v:milti_a(2, KEY_HTLC_REMOTE, KEY_HTLC_LOCAL), 
            or_c(pk(KEY_HTLC_LOCAL), v:hash160(HASH))
        ), 
        older(SELF_DELAY)
    ))
    ``` -->
```
tr(KEY_REVOCATION, { 
    and_v(
        v:milti_a(2, KEY_HTLC_REMOTE, KEY_HTLC_LOCAL), 
        older(SELF_DELAY)), 
    and_v(
        and_v(
            v:pk(KEY_HTLC_REMOTE), 
            v:hash160(HASH)), 
        older(SELF_DELAY)) 
})
```

#### Received HTLC outputs

<!-- single branch version:
    ```
    tr(KEY_REVOCATION, and_v(
        and_v(
            v:multi_a(2, KEY_HTLC_LOCAL, KEY_HTLC_REMOTE), 
            or_c(
                after(CLTV_EXPIRE),
                v:hash160(HASH),
            )),
        older(SELF_DELAY)
    ))
    ``` -->
```
tr(KEY_REVOCATION, {
    and_v(
        and_v(
            v:multi_a(2, KEY_HTLC_LOCAL, KEY_HTLC_REMOTE), 
            after(CLTV_EXPIRE)), 
        older(SELF_DELAY),
    and_v(
        and_v(
            v:multi_a(2, KEY_HTLC_LOCAL, KEY_HTLC_REMOTE), 
            v:hash160(HASH)), 
        older(SELF_DELAY)
})
```

These outputs are constructed after BOLT-3 with `option_anchors` (see rationale).

#### HTLC spending transactions

The input of *HTLC spending transactions* spends *offered HTLC* (for *HTLC timeout transaction*) or *received HTLC* (for *HTLC success transaction*) output from the commitment transaction.

- nLockTime:
  - 0 for HTLC-success
  - `CLTV_EXPIRY` for HTLC-timeout
- Input:
  - nSeq: `SELF_DELAY` (must be even to use P2C deterministic bitcoin commitments)
  - control block: `0xC0/0xC1 <first branch tapleaf-hash>`
  - rest of witness stack for *HTLC-success*:
    - `KEY_HTLC_LOCAL`
    - `SIG(KEY_HTLC_LOCAL)`
    - `KEY_HTLC_REMOTE`
    - `SIG(KEY_HTLC_REMOTE)`
    - `HASH_PREIMAGE`
  - rest of witness stack for *HTLC-timeout*:
    - `KEY_HTLC_REMOTE`
    - `SIG(KEY_HTLC_REMOTE)`
    - `KEY_HTLC_LOCAL`
    - `SIG(KEY_HTLC_LOCAL)`
- Output descriptor:
   ```
   tr(KEY_REVOCATION*, and_v(
       v:pk(KEY_LOCAL_DELAYED),
       older(SELF_DELAY)
   ))
   ```
   (* denotes that the internal taproot key `KEY_REVOCATION` may be tweaked with
   pay-to-contract commitment)

#### On-chain protocol

TBD

## Compatibility

## Rationale

### Use of `OP_CHECKSIGADD`

There are multiple ways to create multisig scheme in Taproot & TapScript [1].
In script-path spending leafs we choose to use `OP_CHECKSIGADD`-based scripts,
defined by miniscript `multi_a` descriptors. This allows to avoid multiple
interactive communications required for composinng aggregated MuSig signatures.
The downside of this approach is a large witness stack size (and higher 
transaction cost), which may be mitigated by adopting PTLC-based scriptless-
script contracts [2].

### Use of `option_anchors`

HTLC construction in Bifrost always requires use of equivalent of legacy BOLT-9
"anchored outputs" option. This addresses a vulnarability caused by the absence 
of self-delay seqnence lock, where local party had an incentive to non-cooperatively
close the channel in order to instantly get access to the local funds.


## Reference implementation

## Acknowledgements

## References

1. <https://github.com/bitcoin/bips/blob/master/bip-0342.mediawiki#cite_note-5>
2. LNPBP-0056

## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.

## Test vectors
