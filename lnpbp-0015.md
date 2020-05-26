```
LNPBP: 0015
Layer: OSI Session (i5)
Vertical: Lightning network protocol
Title: LNP handshake and encryption in network communications based on Noise_XK (BOLT-8 extract)
Author: BOLT-8 and Noise_XK protocol authors
Comments-URI: n/a
Status: Proposal
Type: Standards Track
Created: 2020-12-05
License: CC0-1.0
```

# LNP handshake and encryption in network communications

## Abstract

## Table of Contents

* [Abstract](#abstract)
* [Background](#background)
* [Motivation](#motivation)
* [Design](#design)
    * [Authenticated Key Agreement Handshake](#authenticated-key-agreement-handshake)
    * [Handshake Versioning](#handshake-versioning)
    * [Noise Protocol Instantiation](#noise-protocol-instantiation)
* [Specification](#specification)
    * [Handshake State](#handshake-state)
    * [Handshake State Initialization](#handshake-state-initialization)
    * [Handshake Exchange](#handshake-exchange)
* [Compatibility](#compatibility)
* [Rationale](#rationale)
* [Reference implementation](#reference-implementation)
* [Acknowledgements](#acknowledgements)
* [References](#references)
* [Copyright](#copyright)
* [Test vectors](#test-vectors)

## Background

## Motivation

## Design

Prior to sending any messages, nodes MUST first initiate the cryptographic
session state that is used to encrypt and authenticate all messages sent between
nodes. The initialization of this cryptographic session state is completely
distinct from any inner protocol message header or conventions.

Before any actual data transfer, both nodes participate in an authenticated key agreement handshake, which is based on the Noise Protocol Framework [[2](#ref-2)].

### Authenticated Key Agreement Handshake

The handshake chosen for the authenticated key exchange is `Noise_XK`. As a
pre-message, the initiator must know the identity public key of
the responder. This provides a degree of identity hiding for the
responder, as its static public key is _never_ transmitted during the handshake. Instead,
authentication is achieved implicitly via a series of Elliptic-Curve
Diffie-Hellman (ECDH) operations followed by a MAC check.

The authenticated key agreement (`Noise_XK`) is performed in three distinct
steps (acts). During each act of the handshake the following occurs: some (possibly encrypted) keying
material is sent to the other party; an ECDH is performed, based on exactly
which act is being executed, with the result mixed into the current set of
encryption keys (`ck` the chaining key and `k` the encryption key); and
an AEAD payload with a zero-length cipher text is sent. As this payload has no
length, only a MAC is sent across. The mixing of ECDH outputs into
a hash digest forms an incremental TripleDH handshake.

Using the language of the Noise Protocol, `e` and `s` (both public keys)
indicate possibly encrypted keying material, and `es`, `ee`, and `se` each indicate an
ECDH operation between two keys. The handshake is laid out as follows:
```
    Noise_XK(s, rs):
       <- s
       ...
       -> e, es
       <- e, ee
       -> s, se
```
All of the handshake data sent across the wire, including the keying material, is
incrementally hashed into a session-wide "handshake digest", `h`. Note that the
handshake state `h` is never transmitted during the handshake; instead, digest
is used as the Associated Data within the zero-length AEAD messages.

Authenticating each message sent ensures that a man-in-the-middle (MITM) hasn't modified
or replaced any of the data sent as part of a handshake, as the MAC
check would fail on the other side if so.

A successful check of the MAC by the receiver indicates implicitly that all
authentication has been successful up to that point. If a MAC check ever fails
during the handshake process, then the connection is to be immediately
terminated.

### Handshake Versioning

Each message sent during the initial handshake starts with a single leading
byte, which indicates the version used for the current handshake. A version of 0
indicates that no change is necessary, while a non-zero version indicate that the
client has deviated from the protocol originally specified within this
document.

Clients MUST reject handshake attempts initiated with an unknown version.

### Noise Protocol Instantiation

Concrete instantiations of the Noise Protocol require the definition of
three abstract cryptographic objects: the hash function, the elliptic curve,
and the AEAD cipher scheme. For Lightning, `SHA-256` is
chosen as the hash function, `secp256k1` as the elliptic curve, and
`ChaChaPoly-1305` as the AEAD construction.

The composition of `ChaCha20` and `Poly1305` that are used MUST conform to
`RFC 7539`<sup>[1](#reference-1)</sup>.

The official protocol name for the Lightning variant of Noise is
`Noise_XK_secp256k1_ChaChaPoly_SHA256`. The ASCII string representation of
this value is hashed into a digest used to initialize the starting handshake
state. If the protocol names of two endpoints differ, then the handshake
process fails immediately.

## Specification

The handshake proceeds in three acts, taking 1.5 round trips. Each handshake is
a _fixed_ sized payload without any header or additional meta-data attached.
The exact size of each act is as follows:

   * **Act One**: 50 bytes
   * **Act Two**: 50 bytes
   * **Act Three**: 66 bytes

### Handshake State

Throughout the handshake process, each side maintains these variables:

 * `ck`: the **chaining key**. This value is the accumulated hash of all
   previous ECDH outputs. At the end of the handshake, `ck` is used to derive
   the encryption keys for Lightning messages.

 * `h`: the **handshake hash**. This value is the accumulated hash of _all_
   handshake data that has been sent and received so far during the handshake
   process.

 * `temp_k1`, `temp_k2`, `temp_k3`: the **intermediate keys**. These are used to
   encrypt and decrypt the zero-length AEAD payloads at the end of each handshake
   message.

 * `e`: a party's **ephemeral keypair**. For each session, a node MUST generate a
   new ephemeral key with strong cryptographic randomness.

 * `s`: a party's **static keypair** (`ls` for local, `rs` for remote)

The following functions will also be referenced:

  * `ECDH(k, rk)`: performs an Elliptic-Curve Diffie-Hellman operation using
    `k`, which is a valid private key, and `rk`, which is a `secp256k1` public key
    within the finite field, as defined by the curve parameters
      * The returned value is the SHA256 of the DER-compressed format of the
	    generated point.

  * `HKDF(salt,ikm)`: a function defined in `RFC 5869`<sup>[3](#reference-3)</sup>,
    evaluated with a zero-length `info` field
     * All invocations of `HKDF` implicitly return 64 bytes of
       cryptographic randomness using the extract-and-expand component of the
       `HKDF`.

  * `encryptWithAD(k, n, ad, plaintext)`: outputs `encrypt(k, n, ad, plaintext)`
     * Where `encrypt` is an evaluation of `ChaCha20-Poly1305` (IETF variant)
       with the passed arguments, with nonce `n` encoded as 32 zero bits,
       followed by a *little-endian* 64-bit value. Note: this follows the Noise
       Protocol convention, rather than our normal endian.

  * `decryptWithAD(k, n, ad, ciphertext)`: outputs `decrypt(k, n, ad, ciphertext)`
     * Where `decrypt` is an evaluation of `ChaCha20-Poly1305` (IETF variant)
       with the passed arguments, with nonce `n` encoded as 32 zero bits,
       followed by a *little-endian* 64-bit value.

  * `generateKey()`: generates and returns a fresh `secp256k1` keypair
     * Where the object returned by `generateKey` has two attributes:
         * `.pub`, which returns an abstract object representing the public key
         * `.priv`, which represents the private key used to generate the
           public key
     * Where the object also has a single method:
         * `.serializeCompressed()`

  * `a || b` denotes the concatenation of two byte strings `a` and `b`

### Handshake State Initialization

Before the start of Act One, both sides initialize their per-sessions
state as follows:

 1. `h = SHA-256(protocolName)`
    * where `protocolName = "Noise_XK_secp256k1_ChaChaPoly_SHA256"` encoded as
      an ASCII string

 2. `ck = h`

 3. `h = SHA-256(h || prologue)`
    * where `prologue` is the ASCII string: `lightning`

As a concluding step, both sides mix the responder's public key into the
handshake digest:

 * The initiating node mixes in the responding node's static public key
   serialized in Bitcoin's DER-compressed format:
   * `h = SHA-256(h || rs.pub.serializeCompressed())`

 * The responding node mixes in their local static public key serialized in
   Bitcoin's DER-compressed format:
   * `h = SHA-256(h || ls.pub.serializeCompressed())`

### Handshake Exchange

#### Act One

```
    -> e, es
```

Act One is sent from initiator to responder. During Act One, the initiator
attempts to satisfy an implicit challenge by the responder. To complete this
challenge, the initiator must know the static public key of the responder.

The handshake message is _exactly_ 50 bytes: 1 byte for the handshake
version, 33 bytes for the compressed ephemeral public key of the initiator,
and 16 bytes for the `poly1305` tag.

**Sender Actions:**

1. `e = generateKey()`
2. `h = SHA-256(h || e.pub.serializeCompressed())`
     * The newly generated ephemeral key is accumulated into the running
       handshake digest.
3. `es = ECDH(e.priv, rs)`
     * The initiator performs an ECDH between its newly generated ephemeral
       key and the remote node's static public key.
4. `ck, temp_k1 = HKDF(ck, es)`
     * A new temporary encryption key is generated, which is
       used to generate the authenticating MAC.
5. `c = encryptWithAD(temp_k1, 0, h, zero)`
     * where `zero` is a zero-length plaintext
6. `h = SHA-256(h || c)`
     * Finally, the generated ciphertext is accumulated into the authenticating
       handshake digest.
7. Send `m = 0 || e.pub.serializeCompressed() || c` to the responder over the network buffer.

**Receiver Actions:**

1. Read _exactly_ 50 bytes from the network buffer.
2. Parse the read message (`m`) into `v`, `re`, and `c`:
    * where `v` is the _first_ byte of `m`, `re` is the next 33
      bytes of `m`, and `c` is the last 16 bytes of `m`
    * The raw bytes of the remote party's ephemeral public key (`re`) are to be
      deserialized into a point on the curve using affine coordinates as encoded
      by the key's serialized composed format.
3. If `v` is an unrecognized handshake version, then the responder MUST
    abort the connection attempt.
4. `h = SHA-256(h || re.serializeCompressed())`
    * The responder accumulates the initiator's ephemeral key into the authenticating
      handshake digest.
5. `es = ECDH(s.priv, re)`
    * The responder performs an ECDH between its static private key and the
      initiator's ephemeral public key.
6. `ck, temp_k1 = HKDF(ck, es)`
    * A new temporary encryption key is generated, which will
      shortly be used to check the authenticating MAC.
7. `p = decryptWithAD(temp_k1, 0, h, c)`
    * If the MAC check in this operation fails, then the initiator does _not_
      know the responder's static public key. If this is the case, then the
      responder MUST terminate the connection without any further messages.
8. `h = SHA-256(h || c)`
     * The received ciphertext is mixed into the handshake digest. This step serves
       to ensure the payload wasn't modified by a MITM.

#### Act Two

```
   <- e, ee
```

Act Two is sent from the responder to the initiator. Act Two will _only_
take place if Act One was successful. Act One was successful if the
responder was able to properly decrypt and check the MAC of the tag sent at
the end of Act One.

The handshake is _exactly_ 50 bytes: 1 byte for the handshake version, 33
bytes for the compressed ephemeral public key of the responder, and 16 bytes
for the `poly1305` tag.

**Sender Actions:**

1. `e = generateKey()`
2. `h = SHA-256(h || e.pub.serializeCompressed())`
     * The newly generated ephemeral key is accumulated into the running
       handshake digest.
3. `ee = ECDH(e.priv, re)`
     * where `re` is the ephemeral key of the initiator, which was received
       during Act One
4. `ck, temp_k2 = HKDF(ck, ee)`
     * A new temporary encryption key is generated, which is
       used to generate the authenticating MAC.
5. `c = encryptWithAD(temp_k2, 0, h, zero)`
     * where `zero` is a zero-length plaintext
6. `h = SHA-256(h || c)`
     * Finally, the generated ciphertext is accumulated into the authenticating
       handshake digest.
7. Send `m = 0 || e.pub.serializeCompressed() || c` to the initiator over the network buffer.

**Receiver Actions:**

1. Read _exactly_ 50 bytes from the network buffer.
2. Parse the read message (`m`) into `v`, `re`, and `c`:
    * where `v` is the _first_ byte of `m`, `re` is the next 33
      bytes of `m`, and `c` is the last 16 bytes of `m`.
3. If `v` is an unrecognized handshake version, then the responder MUST
    abort the connection attempt.
4. `h = SHA-256(h || re.serializeCompressed())`
5. `ee = ECDH(e.priv, re)`
    * where `re` is the responder's ephemeral public key
    * The raw bytes of the remote party's ephemeral public key (`re`) are to be
      deserialized into a point on the curve using affine coordinates as encoded
      by the key's serialized composed format.
6. `ck, temp_k2 = HKDF(ck, ee)`
     * A new temporary encryption key is generated, which is
       used to generate the authenticating MAC.
7. `p = decryptWithAD(temp_k2, 0, h, c)`
    * If the MAC check in this operation fails, then the initiator MUST
      terminate the connection without any further messages.
8. `h = SHA-256(h || c)`
     * The received ciphertext is mixed into the handshake digest. This step serves
       to ensure the payload wasn't modified by a MITM.

#### Act Three

```
   -> s, se
```

Act Three is the final phase in the authenticated key agreement described in
this section. This act is sent from the initiator to the responder as a
concluding step. Act Three is executed _if and only if_ Act Two was successful.
During Act Three, the initiator transports its static public key to the
responder encrypted with _strong_ forward secrecy, using the accumulated `HKDF`
derived secret key at this point of the handshake.

The handshake is _exactly_ 66 bytes: 1 byte for the handshake version, 33
bytes for the static public key encrypted with the `ChaCha20` stream
cipher, 16 bytes for the encrypted public key's tag generated via the AEAD
construction, and 16 bytes for a final authenticating tag.

**Sender Actions:**

1. `c = encryptWithAD(temp_k2, 1, h, s.pub.serializeCompressed())`
    * where `s` is the static public key of the initiator
2. `h = SHA-256(h || c)`
3. `se = ECDH(s.priv, re)`
    * where `re` is the ephemeral public key of the responder
4. `ck, temp_k3 = HKDF(ck, se)`
    * The final intermediate shared secret is mixed into the running chaining key.
5. `t = encryptWithAD(temp_k3, 0, h, zero)`
     * where `zero` is a zero-length plaintext
6. `sk, rk = HKDF(ck, zero)`
     * where `zero` is a zero-length plaintext,
       `sk` is the key to be used by the initiator to encrypt messages to the
       responder,
       and `rk` is the key to be used by the initiator to decrypt messages sent by
       the responder
     * The final encryption keys, to be used for sending and
       receiving messages for the duration of the session, are generated.
7. `rn = 0, sn = 0`
     * The sending and receiving nonces are initialized to 0.
8. Send `m = 0 || c || t` over the network buffer.

**Receiver Actions:**

1. Read _exactly_ 66 bytes from the network buffer.
2. Parse the read message (`m`) into `v`, `c`, and `t`:
    * where `v` is the _first_ byte of `m`, `c` is the next 49
      bytes of `m`, and `t` is the last 16 bytes of `m`
3. If `v` is an unrecognized handshake version, then the responder MUST
    abort the connection attempt.
4. `rs = decryptWithAD(temp_k2, 1, h, c)`
     * At this point, the responder has recovered the static public key of the
       initiator.
5. `h = SHA-256(h || c)`
6. `se = ECDH(e.priv, rs)`
     * where `e` is the responder's original ephemeral key
7. `ck, temp_k3 = HKDF(ck, se)`
8. `p = decryptWithAD(temp_k3, 0, h, t)`
     * If the MAC check in this operation fails, then the responder MUST
       terminate the connection without any further messages.
9. `rk, sk = HKDF(ck, zero)`
     * where `zero` is a zero-length plaintext,
       `rk` is the key to be used by the responder to decrypt the messages sent
       by the initiator,
       and `sk` is the key to be used by the responder to encrypt messages to
       the initiator
     * The final encryption keys, to be used for sending and
       receiving messages for the duration of the session, are generated.
10. `rn = 0, sn = 0`
     * The sending and receiving nonces are initialized to 0.

## Compatibility

## Rationale

## Reference implementation

## Acknowledgements

## References

1. <a id="ref-1">&nbsp;</a>
   BOLT-8: Encrypted and Authenticated Transport. Version 1.
   <https://github.com/lightningnetwork/lightning-rfc/blob/v1.0/08-transport.md>
2. <a id="ref-2">&nbsp;</a>
   Trevor Perrin. The Noise Protocol Framework. Revision 34, 2018-07-11.
   <http://noiseprotocol.org/noise.pdf>

## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.

## Test vectors
