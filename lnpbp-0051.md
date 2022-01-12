```
LNPBP: 0051
Vertical: Lightning network protocol
Title: Bifrost: channel management protocol
Author: Dr Maxim Orlovsky <orlovsky@lnp-bp.org>
Comments-URI: https://github.com/LNP-BP/lnpbps/pulls/97
Status: Draft
Type: Standards Track
Created: 2021-11-01
Finalized: not yet
License: CC0-1.0
```

- [Abstract](#abstract)
- [Background](#background)
- [Motivation](#motivation)
- [Design](#design)
- [Specification](#specification)
  - [Bifrost transaction requirements](#bifrost-transaction-requirements)
  - [Channel coordination](#channel-coordination)
    - [Channel workflows](#channel-workflows)
- [Channel creation workflow](#channel-creation-workflow)
- [Compatibility](#compatibility)
- [Rationale](#rationale)
- [Reference implementation](#reference-implementation)
- [Acknowledgements](#acknowledgements)
- [References](#references)
- [Copyright](#copyright)
- [Test vectors](#test-vectors)


## Abstract

## Background

## Motivation

## Design

## Specification

### Bifrost transaction requirements

Bifrost requires all off-chain transactions always be of version 2. Transaction
outputs which are internal to the channel (i.e. spend by other offchain
transactions participating in channel) MUST use v1 witness P2TR outputs (or
later witness versions). The scripts in taproot key path spends MUST be
miniscript-compatible.

For on-chain funding transactions and funding outputs of channel level 1 this
requirement is released to witness v0 or above. The reason for lower requirement
is the interoperability with the legacy lightning network, allowing migration of
existing channels opened in legacy network to Bifrost.

There is no specific requirements for outputs which are not internal for the
channel.

### Channel coordination

For channel operations we assume that any channel may be a multi-peer channel.
Thus, for channel updates it is required that all parties cooperate and sign the
latest version of updated channel transactions. This is achieved by introducing
concept of *channel coordinator*. Channel coordinator is the lightning node that
has originally proposed channel. It is responsible for orchestrating message
flow between all nodes which are the parts of the channel and keeping them
up-to-date. Also, the channel coordinator is the only party required to have
direct connections with other channel participants – and each of channel
participants is required to be connected at least to the channel coordinator.

If a multiple nested channels are present, for all higher-level channels channel
coordinator MUST be the same as channel coordinator for the base (level 1)
channel; the list of participants for the nested channels MUST be a subset of
the participants of the topmost level 1 channel.


#### Channel workflows

There are following workflows affecting channel status / existence. Each of
these workflows represent a set of P2P messages exchanged by channel peers.

- Channel creation
- Moving channel from legacy to Bifrost LN
- Removing channel from Bifrost to legacy LN
- Changing channel status (pausing etc)
- Upgrading channel to support more protocols
- Downgrading channel by removing specific protocol
- Cooperatively closing channel

Workflow can be initiated only by a *channel coordinator*, and specific P2P
messages inside the workflow can be sent either from the *channel coordinator*
to a peer – or, in response, from a peer to the *channel coordinator*.

Normal channel operations are covered by application-specific business logic and
messages and are not part of any listed channel workflow. Unlike workflows, they
may be initiated by any of the channel peers sending message to the channel
coordinator, however whenever they involve other peers or external channels,
after being initiated they must be coordinated by the channel coordinator.

## Channel creation workflow

Considering generic case of multi-peer channel setup channel creation workflow
is organized with the following algorithm:

1. First, all parties agree on the structure of the *funding transaction*
   and overall transaction graph within the channel – simultaneously signing
   *refund transaction* (which, upon channel creation, will become first
   version of the channel *commitment transaction*). This is done using
   `ProposeChannel` requests sent by the *channel coordinator* to each of
   the peers, replying with either `AcceptChannel` (containing updated
   transaction graph with signed refund transaction) or `Error`.
   peers must wait for `CHANNEL_CREATION_TIMEOUT` period and discard all
   provisional channel data from their memory.

2. Once the refund transaction is fully signed – implying that the
   transaction graph if agreed between participants – channel coordinator
   starts next phase, where the funding transaction gets fully signed.
   Coordinator sends `FinalizeChannel` message to each of the peers and
   collects signatures, publishing the final transaction either to bitcoin
   blockchain (for level 1 channels) or updating the state of the top-level
   channel (for nested channels above level 1). Peers track upper level
   channel or blockchain to detect funding transaction, and upon transaction
   mining starts operate channel in active mode, not requiring any other
   messages from the channel coordinator (NB: this differs from the legacy
   LN channel creation workflow).

3. Replacing funding by fee (RBF): channel coordinator SHOULD initiate RBF
   subworkflow for level 1 channels if the funding transaction was not mined
   after reasonable amount of time, which should be less than
   `ChannelParams::funding_timeout`. With RGB subworkflow coordinator
   updates funding transaction – and propagates it with `FinalizeChannel`
   request, collecting new signatures (peers MUST reset their funding
   timeout counters).

4. Cancelling channel creation: if any of the peer nodes replied with
   `Error` on any of the channel construction requests within the channel
   creation workflow – or if the coordinator detected incorrect reply,
   channel coordinator MUST abandon channel creation – and MUST forward
   `Error` message to all other peers. A peer posting `Error` MUST
   provide a valid error code and a message explaining the cause of the
   error. The coordinator SHOULD also send `Error` message to peers if
   any of the stages of transaction construction workflow has stuck
   without a reply from a peer for over `ChannelParams::peer_timeout`
   time.

5. Timeouts: the coordinator SHOULD send `Error` message to peers if any
   of the peers at any stage of transaction construction workflow has stuck
   without a reply for over `ChannelParams::peer_timeout` time.
   The peers should abandon channel and clear all information about it from
   the memory regardless whether they have received `Error` message from
   the coordinator after `ChannelParams::peer_timeout`` * 2` time before
   `ChannelFinalized` – and if they has not received new
   `FinalizeChannel` request from the coordinator after
   `ChannelParams::funding_timeout` time (see pt 3 for RBF subworkflow).

```
Channel coordinator                   Peer 1             Peer 2
        |                               |                  |
(enters ChannelProposed state)          |                  |
        |                               |                  |
        | --(1)- ProposeChannel ------> |                  |
        |                               |                  |
        | --(1)------------ ProposeChannel --------------> |
        |                               |                  |
        |                           (enter ChannelProposed state)
        |                               |                  |
        | <-(2)------------- AcceptChannel --------------- |
        |                               |                  |
        | <-(2)-- AcceptChannel ------- |                  |
        |                               |                  |
 (enters ChannelAccepted state)     (enter ChannelAccepted state)
        |                               |                  |
        | --(3)- FinalizeChannel -----> |                  |
        |                               |                  |
        | --(3)------------ FinalizeChannel -------------> |
        |                               |                  |
        | <-(4)-- FinalizeChannel ----- |                  |
        |                               |                  |
        | <-(4)------------- FinalizeChannel ------------- |
        |                               |                  |
 (enters ChannelFinalized state)    (enter ChannelFinalized state)
        |                               |                  |
(await funding transaction mining or entering the valid super-channel state)
        |                               |                  |
 (enters ChannelActive state)       (enter ChannelActive state)
        |                               |                  |
```

During channel construction workflow channels are identified by
`ChannelId`, which is constructed as a tagged SHA-256 hash
(using `bifrost:channel-proposal` as tag) of the strict-serialized
`ChannelParams` data and coordinator node public key.

## Compatibility

## Rationale

## Reference implementation

## Acknowledgements

## References

## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.

## Test vectors
