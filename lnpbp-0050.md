```
LNPBP: 0050
Layer: OSI Application (i7)
Vertical: Lightning network protocol
Title: Bifrost: generalized Lightning network protocol core
Author: Dr Maxim Orlovsky <orlovsky@lnp-bp.org>
Comments-URI: https://github.com/LNP-BP/lnpbps/pulls/<____>
Status: Draft
Type: Standards Track
Created: 2021-11-01
Finalized: not yet
License: CC0-1.0
```

## Abstract


## Background


## Motivation

Extending existing lightning network messaging system, defined as part of
multiple BOLTs, is hard. The first problem is the necessity to introduce changes
to multiple independent standards for each atomic feature; making review and
acceptance process very time-consuming. Next, it is hard to implement these
features one by one, since they are defined in different places. Finally, there
is no place to put feature-specific test vectors in the existing BOLT structure.

The second problem is separation of new and not yet well tested/experimental
functionality from the well-tested existing lightning network core. The
separation of experimental protocols and existing tested core functionality is
good from the security point of view.

The third reason is the fact that Lightning network was designed as a payment
network and for payment purposes. At the same time, it is possible to create
much more advanced forms of non-payment state channels basing on bitcoin
transactions (useful for storage and different forms of financial smart
contracts), or use lightning network for non-payment data communication (like
messaging or DEX). These extensions when put into existing LN spec framework,
not suited for such extensibility, will require constant introduction of
multiple hacks or complete refactoring of the BOLT specifications.

Finally, some of lightning network design decisions were proven to be
non-efficient in context of more broad applications, for instance use of BigInt
encoding is not advised for client-side-validated data […]. It will be
impossible to fix such issues without introducing new network communication
protocol.

Bifrost is a proposed new set of standards, defined as a part of LNPBP standards
suite, that includes extensible core networking protocol (LNPBP-50, this
specification), and specific extensions for different forms of state channel
management and network data communications. Current standard is built on top of
other LNPBP standards (like strict encoding), BIPs (partially signed bitcoin
transactions), parts of BOLT standards (Noise_XK protocol), extracted as a
separate LNPBP standards, and puts them into a single well-abstracted protocol
suite.


## Design

Design goals:
1. Extensibility, including
    - support for non-payment / custom lightning channels
    - support for non-payment network messaging & communications
    - support for arbitrary extension of channel transaction structure
      (combining different types of channels)
    - support for custom / new route discovery mechanisms
2. Maximal use of existing LNPBP standards, in particular
    - LNPBP-7 strict encoding […]
    - LNPBP-9 client-side-validation […]
    - LNPBP-15 Noise_XK handshake & network encryption (BOLT-8 extract) […]
    - LNPBP-18 Native message framing (BOLT-8 extract) […]
    - LNPBP-16 handshake over WebSockets […]
3. Privacy: improved re-use of onion messaging from Lightning network


## Specification

Bifrost operates using TCP connection, on top of LNPBP-15 (session & encryption
layer) and LNPBP-18 (message framing layer) as application protocol. Default
port for Bifrost connection is 9913 (see [rationale](#use-of-dedicated-port)).

All message fields are encoded with LNPBP-7 strict encoding instead of other
types of encoding used in BOLTs / legacy Lightning network.

### Announcing Bifrost connectivity

Support for Bifrost protocol can be announced as a part of local features flag
([see why not global](#using-local-features)) in legacy lightning network `init`
message, inside `node_announcement`, `channel_announcement` and `channel_update`
messages (`INC` context in terms of BOLT-9). Bifrost feature flags should not be
used with BOLT-11 invoices; instead, all Bifrost channels MUST use LNPBP-38
invoicing.

Since BOLT-9 provides no ability to add arbitrary feature flags from outside of
the BOLT-defined protocol scope, this feature will be a non-standard. We reserve
flag number 255/256 for it (see [rationale](#feature-bit-selection)).


### Bifrost URLs

Bifrost URLs are designed to be compliant with RFC 3986 [1] and consists of
`bifrost` as *scheme name*, *authority* in form of node public key, combined
with optional network address details, channel id as a *path* and feature
parameters and route suggestion as *query*, where both *path* and *query* parts
are optional (emphasized words stand for RFC 3986-defined terms):

`<schema>://<authority>[/<path>][?<query>]` =>
`bifrost://<host>[/<channel-id>][?<feature-params> | <route-suggestion>]`

Host part MUST always include node public key, optionally followed by either
IPv4, IPv6 or ONION v3 Tor address. IPv4 and IPv6 addresses may include optional
port number, which becomes mandatory if a non-standard Bifrost port is used. The
address, if present, MUST be separated from the public key by `+` symbol (see
[rationale](#why-URL-uses-plus-instead-of-at-symbol)).

`bifrost://<key>[+ <IPv4 | IPv6>[:<port>] | <ONIONv3>]/[<channel-id>][?<feature-params> | <route-suggestion>]`

Example:

- Node with a known IPv4 address operating non-standard port:
  `bifrost://02b39e7040cd9233e1cf86823ea321c1c799534c622c9e8b563a689409962657c7+124.155.54.210:9935`

- Node with Onion v3 address and path suggestion:
  `bifrost://02b39e7040cd9233e1cf86823ea321c1c799534c622c9e8b563a689409962657c7+worldehc62cgugrgj7oc76tcna45fme47oqjrei4d4aa7xorw7fyvcyd/?`


### Bifrost initiation from legacy Lightning messaging

Two nodes connected via Lightning network may send each other specific message
indicating intent to open Bifrost connection. The message uses onion routing,
enabling NAT bypassing, such that a party without NAT may “hole punch”  NAT of
another party by asking to establish Bifrost connection to its explicit IP
address.

### Establishing connection




## Compatibility

### Channels

Bifrost may be used to operate channels created using legacy Lightning network
protocols – until there are changes into the channel structure which make them
incompatible with existing BOLT specifications. Also, channels created with
Bifrost can be accessed via legacy lightning network until they become
incompatible (in their structure) with BOLT specifications.

Thus, to prevent undefined behavior, it is advised that nodes will maintain each
channel either under legacy LN or under Bifrost using the following rules:

1. If the channel was created in legacy LN it can be moved into Bifrost once and
   only once – and a dedicated gossip `channel_update` message must be published
   to the legacy LN with Bifrost even flag set.

2. If the channel was created in Bifrost network, it MAY be  announced in the
   legacy LN only with odd (required) Bifrost flag set.

3. Bifrost channels can’t be operated using legacy LN messaging; a node
   receiving message referencing to a Bifrost server over the legacy LN MUST
   respond with `error` message.

4. Legacy LN channels MUST not be operated from Bifrost network before their
   movement according to the pt. 1. A node receiving Bifrost message for legacy
   LN channel MUST respond with `error` message.

   In order to enable fallback for Bifrost-managed channels in case of bugs
   discovered in Bifrost, the following rules SHOULD be supported by Lightning
   node implementations, but SHOULD not be used until a fallback scenario:

5. Bifrost channels which maintain BOLT compatibility can be moved back into
   legacy Lightning network by properly announcing removal of the channel in
   Bifrost network – and publishing `channel_announcement` (for previously
   non-announced channels) or `channel_update` gossip messages with all Bifrost
   feature flags removed

### Invoices

All bifrost-operated channels MUST use LNPBP-38 invoices. BOLT-11 invoices MUST
NOT be created for any channel which was moved into Bifrost network.


## Rationale

### Use of dedicated port

Bifrost requires use of a non-LN port to enable simultaneous and independent
operations of lightning nodes using both legacy LN messages and Bifrost
messages.

### Why URL uses plus instead of at symbol

While at* symbol (`@`) is broadly used in legacy Lightning node addresses
(`<pubkey>@<host>[:<port>]` form), it can’t be used as a proper part of Bifrost
URL since in RFC 3986 it is reserved as a separator for optional username prefix
before mandatory host name part – while pubkey is not optional in Bifrost and
host is optional. Thus, we replace `@` usage with `+`, since `+` is allowed in
RFC 3986 as a separator in host name part of authority field and it semantically
defines “additional information”, which is a network address of the node for a
given public key.

### Using local features

Bifrost support is specified in the local features field of the `init` message
since it is against BOLT-9 to use feature bits greater than 13 in the global
features field.

### Feature bit selection

Bits 255/256 are the largest bit numbers which may be stored by a 32-byte value.
It is still far enough from the current used feature bits in BOLT-9 (26/27),
allowing another 5 to 10 years without the risk of the conflict (until Bifrost
will be recognized) and at the same time does not over-enlarges the size of the
feature flag field in `init` and other messages.


## Reference implementation

The current Bifrost reference implementation is provided by [`LNP Core
Library`](https://github.com/LNP-BP/lnp-core) and [`LNP
Node`](https://github.com/LNP-BP/lnp-node)

## Acknowledgements

The authors are thankful to Giacomo Zucco, Christian Decker and Martin
Habovštiak for multiple discussions and ideas which led to the creation and
shaping of this specification.


## References

1. RFC 3986. Uniform Resource Identifier (URI): Generic Syntax. https://www.rfc-editor.org/rfc/rfc3986#section-3.1

## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.


## Test vectors

TBD
