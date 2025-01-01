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

This specification proposes a batch of improvements for lightning payment 
channels: (1) generalization of channels to have an arbitrary number of 
participants (multi-peer channels), (2) abstraction from specific peyment
routing syncrhronization algorithm (HTLC, PTLC), (3) ability to extend
and change channel parameters and structure on-the-go, (4) use of Schnorr
signatures, MuSig, Taproot and miniscript-based descriptors. It achieves
this properties by leveraging Bifrost protocol, specifically LNPBP-50 and
LNPBP-51 standards.

## Background

Lightning network enables scalable and more private bitcoin payments by
using pre-signed off-chain transactions, composing so-called state channels.
Still, existing lightning network, defined by a set of BOLT standards, has
multiple limitations. LNP/BP standards LNPBP-50, 51, 52 proposed an extended
version of the lightning network called Bifrost. It is a part of more general
Lightning network protocol and allows creation of more complex, functional
and secure forms of payment channels, which is the aim of this proposal.


## Motivation

Current standards defining lightning network (BOLTs) were created in the
eraly SegWit era. They were created to cover single use case (unilaterally
funded dual party payment channels) with a very restricted functionality
(no channel splits, channel factories, nested channels, extensions for
channel structure etc). Since LN launch a multiple enhancemend had been
deployed on Bitcoin maiinnet, including Shnorr signatures, unlocking
more compact multisigs (with MuSig scheme), scriptless scripts & adaptor 
signatures, enabling alternative payment routing synchronization 
mechanism (PTLC), replacing vulnerable HTLC; Taproot, enabling stronger 
privacy properties and more compact witnesses; client-side-validation
applications, including RGB smart contracts; different complex Bitcoin
finance applications (BiFi), like descrete log contracts, lightspeed protocol
and many others. The current standard is aiming at generalizing concept
of lightning payment channels as a part of LNP (lightning network protocol)
family of standards, leveraging new set of functionality offered by
Bifrost protocol.


## Design

First, we separate the core of the payment channel operations from other
channel-related protocols, like payment routing. This enables us real-time
upgradability and composability for channel structure and use of different
protocols within the scope of the same channel (like HTLC, PTLC etc).

Second, we update channel transactions to use Taproot, Schnorr signatures,
MuSig, TapScript – and enable sign-to-contract and pay-to-contract commitments
required for client-side-validation (like RGB smart contracts).

Third, we generalize the structure of payment channel transactions for an
arbitrary number of participants.

Finally, we re-define a set of messages controlling channel creation and 
oprations using channel proposals, strict encoding and other advancements
brought by un underlying Bifrost protocol (LNPBP-50 and 51).


## Specification

### Funding transaction

Transaction containing multiple inputs, each of which MUST signal S2C commitment type.

Output descriptor: `tr(musig(KEY_LOCAL_FUNDING, KEY_REMOTE_1_FUNDING, KEY_REMOTE_2_FUDING, ...))`


### Commitment transaction

Input:
- Spends funding output
- nSeq: `0x80800000 + COMMITMENT_NO & 0xFFFF` (signals S2C DBC)
- Witness stack:
  - `KEY_LOCAL_FUNDING + KEY_REMOTE_1_FUNDING + KEY_REMOTE_2_FUNDING + ...`
  - `MUSIG*(KEY_LOCAL_FUNDING + KEY_REMOTE_1_FUNDING + KEY_REMOTE_2_FUNDING + ...)`
  where * denotes optional presence of sign-to-contract commitment

Outputs:
- `to_local`: `tr(KEY_REVOCATION, and_v(v:pk(KEY_LOCAL),older(SELF_DELAY)))`
  * `KEY_LOCAL`: proposed by the local node
  * `SELF_DELAY`: proposed by the remote node, may be rejected by local
- `to_remote_n`: `tr(KEY_UNSPENDABLE, and_v(v:pk(KEY_REMOTE_N),older(REMOTE_N_DELAY))`
  * `KEY_REMOTE_N`: propsed by the remote node N
  * `KEY_UNSPENDABLE`: a deterministically-unspendable key computed as `KEY_REMOTE_N + SHA256T("lnpbp53:unspendable", KEY_REMOTE_N) * G`
  * `REMOTE_N_DELAY`: proposed by the channel coordinator, may be rejected by remote node N
- `anchor`: `tr(KEY_UNSPENDABLE, {v:pk(KEY_LOCAL), v:pk(KEY_REMOTE_N)})`
  * The ooutput must contain a fixed amount of satoshis above dust limit proposed by
    the node adding this output to the channel proposal

### Messaging

TBD


## Compatibility

### Upgrade

Any legacy lightning network channel can be upgraded into Bifrost channel using
`UPGRADE` Bifrost message.

### Downgrade

Channel may be downgraded from Bifrost to legacy Lightning network if and only if
- Channel is a 2-peer channel
- It contains only HTLC extensions and no other types of extensions
- It is not used in creation of other channels
- It originates from an on-chain funding transaction
- It does not contain RGB assets or other protocols requiring use of
  deterministic bitcoin commitments or single-use-seals

Bifrost channel breaking one of these requirements may be made downgradable by
removing channel protocols until all of the the requirements are met.

Channel resulting from downgrade will always use `anchor_output` BOLT-9 feature
flag.

## Rationale

### Funding transaction inputs use sign-to-contract scheme

With pay-to-contract commitments if any of channel participants have the same 
RGB asset or other P2C-committed protocol data they will be required to merge 
their state transitions, expose the private asset data for all other 
participants and exchange full RGB consignments before signing funding 
transaction. This creates significant privacy leak, more attach vectors and
requires much more complex protocol design.

### Channel downgrade

Bifrost is a highly experimental protocol. Ability to downgrade channel to
legacy stable lightning network provides a failback mechanism for cases
when an attack on Bifrost channels was discovered and participating parties
do not want to close the channel.


## Reference implementation

## Acknowledgements

## References

## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.

## Test vectors
