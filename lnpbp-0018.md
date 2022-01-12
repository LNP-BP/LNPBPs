```
LNPBP: 0018
Vertical: Lightning network protocol
Title: LNP native message framing protocol (BOLT-8 extract)
Author: BOLT-8 protocol authors
Comments-URI: n/a
Status: Proposal
Type: Standards Track
Created: 2020-14-05
License: CC0-1.0
```

- [Abstract](#abstract)
- [Background](#background)
- [Motivation](#motivation)
- [Design](#design)
  - [Encrypting and Sending Messages](#encrypting-and-sending-messages)
  - [Receiving and Decrypting Messages](#receiving-and-decrypting-messages)
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

At the conclusion of Act Three of [LNPBP-15](lnpbp-0015.md), both sides have
derived the encryption keys, which will be used to encrypt and decrypt messages
for the remainder of the session.

The actual Lightning protocol messages are encapsulated within AEAD ciphertexts.
Each message is prefixed with another AEAD ciphertext, which encodes the total
length of the following Lightning message (not including its MAC).

The *maximum* size of _any_ Lightning message MUST NOT exceed `65535` bytes. A
maximum size of `65535` simplifies testing, makes memory management easier, and
helps mitigate memory-exhaustion attacks.

In order to make traffic analysis more difficult, the length prefix for all
encrypted Lightning messages is also encrypted. Additionally a 16-byte
`Poly-1305` tag is added to the encrypted length prefix in order to ensure that
the packet length hasn't been modified when in-flight and also to avoid creating
a decryption oracle.

The structure of packets on the wire resembles the following:

```
+-------------------------------
|2-byte encrypted message length|
+-------------------------------
|  16-byte MAC of the encrypted |
|        message length         |
+-------------------------------
|                               |
|                               |
|     encrypted Lightning       |
|            message            |
|                               |
+-------------------------------
|     16-byte MAC of the        |
|      Lightning message        |
+-------------------------------
```

The prefixed message length is encoded as a 2-byte big-endian integer, for a
total maximum packet length of `2 + 16 + 65535 + 16` = `65569` bytes.

### Encrypting and Sending Messages

In order to encrypt and send a Lightning message (`m`) to the network stream,
given a sending key (`sk`) and a nonce (`sn`), the following steps are
completed:

1. Let `l = len(m)`.
    * where `len` obtains the length in bytes of the Lightning message
2. Serialize `l` into 2 bytes encoded as a big-endian integer.
3. Encrypt `l` (using `ChaChaPoly-1305`, `sn`, and `sk`), to obtain `lc`
    (18 bytes)
    * The nonce `sn` is encoded as a 96-bit little-endian number. As the
      decoded nonce is 64 bits, the 96-bit nonce is encoded as: 32 bits
      of leading 0s followed by a 64-bit value.
        * The nonce `sn` MUST be incremented after this step.
    * A zero-length byte slice is to be passed as the AD (associated data).
4. Finally, encrypt the message itself (`m`) using the same procedure used to
    encrypt the length prefix. Let encrypted ciphertext be known as `c`.
    * The nonce `sn` MUST be incremented after this step.
5. Send `lc || c` over the network buffer.

### Receiving and Decrypting Messages

In order to decrypt the _next_ message in the network stream, the following
steps are completed:

1. Read _exactly_ 18 bytes from the network buffer.
2. Let the encrypted length prefix be known as `lc`.
3. Decrypt `lc` (using `ChaCha20-Poly1305`, `rn`, and `rk`), to obtain the size of
    the encrypted packet `l`.
    * A zero-length byte slice is to be passed as the AD (associated data).
    * The nonce `rn` MUST be incremented after this step.
4. Read _exactly_ `l+16` bytes from the network buffer, and let the bytes be
    known as `c`.
5. Decrypt `c` (using `ChaCha20-Poly1305`, `rn`, and `rk`), to obtain decrypted
    plaintext packet `p`.
    * The nonce `rn` MUST be incremented after this step.

## Compatibility

## Rationale

## Reference implementation

## Acknowledgements

## References

1. <a id="ref-1">&nbsp;</a>
   BOLT-8: Encrypted and Authenticated Transport. Version 1.
   <https://github.com/lightningnetwork/lightning-rfc/blob/v1.0/08-transport.md>

## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.

## Test vectors
