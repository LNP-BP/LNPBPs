```
LNPBP: 0001
Layer: Transactions (1)
Vertical: Bitcoin protocol
Title: Key tweaking: collision-resistant elliptic curve-based commitments
Authors: Dr Maxim Orlovsky <orlovsky@protonmail.ch>,
         Dr Rene Pickhardt,
         Federico Tenga,
         Martino Salvetti,
         Giacomo Zucco,
         Max Hillebrand,
         Christophe Diederichs
Comments-URI: <https://github.com/LNP-BP/lnpbps/issues/3>
Status: Proposal
Type: Standards Track
Created: 2019-09-23
Finalized: not yet
License: CC0-1.0
```

## TOC

- [Abstract](#abstract)
- [Background](#background)
- [Motivation](#motivation)
- [Specification](#specification)
  * [Commitment procedure](#commitment-procedure)
  * [Verification procedure](#verification-procedure)
  * [Reveal procedure](#reveal-procedure)
- [Compatibility](#compatibility)
- [Rationale](#rationale)
- [Reference implementation](#reference-implementation)
- [Acknowledgements](#acknowledgements)
- [References](#references)
- [License](#license)
- [Appendix A. Test vectors](#test-vectors)
  * [Correct test vectors](#correct-test-vectors)
  * [Invalid test vectors](#invalid-test-vectors)
  * [Edge cases: protocol failures](#edge-cases-protocol-failures)


## Abstract

Cryptographic commitments embedded into bitcoin transactions is a widely-used
practice. It's application include timestamping [1], single-use seals [2],
pay-to-contract settlement schemes [3], sidechains [4], blockchain anchoring
[5], Taproot, Graftroot proposals [6, 7, 8], Scriptless scripts [9] and many
others. Nevertheless, existing ways of creating commitments have never been
standardized with best practices and do not commit to the exact protocol or
commitment scheme used. They are also inapplicable to situations where multiple
public keys are present in some output: how to deterministically detect which
key is holding the commitment.

This work proposes a standardization for cryptographic commitments that utilize 
the homomorphic properties of the `Secp256k1` elliptic curve (EC) and allows to 
commit to arbitrary data using an EC public key or a set of EC public keys 
from the `Secp256k1` curve in a deterministic and safe way.


## Background

Cryptographic commitments represent a way to commit to some message without
revealing it. The procedure consists of two phases, **commit** and **reveal**.
In the *commit* phase, a party (**committer**), willing to prove its knowledge
of some message, computes a *cryptographic hash function* over that message
producing a message **digest**, which can be provided to other party(ies). In
the *reveal* phase, the *committer* reveals the actual message and each party
accessing it may check that its hash is equal to the originally provided
*digest*.

Key tweaking is a procedure for creation of a cryptographic commitment to some 
**message** using elliptic curve properties. The procedure uses the discrete log
problem (DLP) as a proof of existence & knowledge of certain information about
the message by some party (Alice) without exposing the original message. This is
done by adding to a public key, for which Alice knows the corresponding private 
key, a hash of the message multiplied on the generator point `G` of the elliptic
curve. This produces a **tweaked** public key, containing the commitment. At
a later time Alice may prove her past knowledge of the original message (at the
time when the commitment was created) by providing a signature corresponding to
the original public key and the message itself.

The main advantage of the public key tweak procedure is the fact that a tweaked 
key, or a corresponding signature, can't be distinguished from any other public 
keys or signatures; this property allows to hide the actual commitment in such
a way that it can only be known to those parties which have knowledge of the 
secrets: the original public and/or key pair **and** a message.

This type of commitment was originally proposed as a part of the *pay to
contract* concept by Ilja Gerhardt and Timo Hanke in [1] and later used by
Eternity Wall [2] for the same purpose. However, these proposals were arguably
vulnerable to length-extension attacks and, more importantly, were not
applicable to scenarios when multiple public keys are used (for instance,
multi-signature bitcoin transaction outputs). These problems were fixed as a
part of the sidechain-design efforts by Blockstream [3], which proposed to
utilize a HMAC function and also introduced a nonce in the concept.

Here we propose a standardization of the algorithm based on the original
Eternity Wall and Blockstream work, enhanced with Pieter Wuille's Tagged Hashes
procedure, coming from a specification on Schnorr signatures in Bitcoin [4],
also used in the Taproot proposal [5]. This procedure prevents cross-protocol
collisions, such that the original message's byte sequence can't be
reinterpreted under another protocol.

## Motivation

Publication of cryptographic commitments to the Bitcoin blockchain is a widely
used mechanism, allowing timestamping of the commitment: it can be used to prove
the fact that some information was known before a certain period in time without
revealing the actual information. Use of elliptic curve homomorphic properties
allows to perform such commitments without increasing the size of the
transaction, by leveraging existing transaction outputs and not polluting
blockchain space with excessive OP_RETURNs. However, as of today, there is no
single standard for such commitments. While different practices for that purpose
exist (see [1, 2, 3]), they contain multiple collision risks, such as the
possibility of length-extension attacks and cross-protocol replay attacks. Or
they can't be applied in situations where multiple public keys are used (
multi-signature or custom bitcoin scripts). This standard combines existing best
practices into a single algorithm, that avoids all of those issues.


## Specification

### Commitment procedure

For a given message `msg`, a list of public keys from the `Secp256k1` curve 
`P* := {P1, P2, ..., Pn}`, `n > 0`, with some selected original public key `Po` 
from this list (`Po ∈ S`), and a protocol-specific `tag` known to both parties, 
the **commit procedure** runs as follows:

1. Reduce list `P*` to a set of unique public keys `P`, by removing all 
   duplicate public keys from the list.
2. Compute sum `S` of all unique public keys in set `P`; fail the protocol if
   an overflow over elliptic curve generator point order happens during the 
   procedure.
3. Construct a byte string `lnpbp1_msg`, composed of the original message
   prefixed with a single SHA256 hash of `LNPBP1` string and a single SHA256
   hash of the protocol-specific `tag`:  
   `lnpbp1_msg = SHA256("LNPBP1") || SHA256(tag) || msg`
4. Serialize the aggregated public key `S` into a 64 byte array `S*` of 
   uncompressed coordinates x and y in big-endian order and use `S*` to
   authenticate `lnbp1_msg` via HMAC-SHA256. The resulting value is named 
   the **tweaking factor** `f`:  
   `f = HMAC-SHA256(lnpbp1_msg, S*)`
5. Make sure that the tweaking factor is less than the order `n` of a generator 
   point of the used elliptic curve, such that no overflow can happen when it is 
   added to the original public key. If the order is exceeded, fail the protocol
   indicating the reason of failure.
6. Multiply the tweaking factor `f` on the used elliptic curve generator point 
   `G`: `F = G * f`
7. Check that the result of step 6 is not equal to the point-at-infinity; 
   otherwise fail the protocol, indicating the reason of failure, such that 
   the protocol may be run with another initial public key set `P'`.
8. Add the two elliptic curve points: the original public key `Po` and the
   point `F`, derived from the tweaking-factor. This will result in a tweaked 
   public key `T`: `T = Po + F`. Check that the result is not equal to the 
   point-at-infinity of the elliptic curve or fail the protocol otherwise, 
   indicating the reason of failure, such that the protocol may be run with 
   another initial public key list `P*'`.

The final formula for the commitment is:  
`T = Po + G * HMAC-SHA256(SHA256("LNPBP1") || SHA256(tag) || msg, S*)`

### Verification procedure

**Verification procedure** for the commitment (i.e. tweaked public key `T`) can 
be performed with the provision of the list of public keys `P*`, 
the original public key `Po` and the message `msg` 
(assuming that the verifying party is aware of the protocol-specific `tag`
and `LNPBP1` tag) and runs as follows:

1. Make sure that the provided tweaked public key `T` lies on the elliptic curve 
   and is not equal to the point at infinity.
2. Compute 
   `T' = Po + G * HMAC-SHA256(SHA256("LNPBP1") || SHA256(tag) || msg, S*)` 
   repeating the *commitment procedure* according to the rules above.
3. Make sure that `T' = T` and report verification success; otherwise report 
   verification failure.

### Reveal procedure

Thus, **reveal data** required for the commitment verification constists of:

1. Original message `msg`
2. Tweaked public key value `T`
3. Original set of public keys `P` and a key `Po` from that set.

The used protocol tag `tag` must be known to all parties participating in the 
protocol.


## Compatibility

The proposed procedure should be compatible with previously-created
pay-to-contract-style commitments based on SHA256 hashes under the assumption of
SHA256 collision resistance. Utilization of a double tagged hash protocol prefix
guarantees randomness in the first 64 bytes of the resulting tweaking string
`lnpbp1_msg`, reducing probability for these bytes to be interpreted as a
correct message under any of the previous standards.

The procedure is well compliant with Taproot SegWit v1, since it operates with
a sum of the original public keys, and the Taproot intermediate key is a sum of
all used public keys, so it can represent a correct input for the protocol.

The tweaked procedure may result in a public key that may, or may not have its 
*y* coordinate being a quadratic residue (in terms of BIP-340 [4]). This may 
present a compatibility issue for using this scheme in 
Taproot/Schnorr-enabled outputs and protocols. Nevertheless, this issue may 
be mitigated by running the procedure a second time and replacing the 
original public key with its own negation, if the resulting tweaked version 
was not square.

The proposal relies on a tagged hash prefix similar to the one used in 
BIP-340, [4], which helps to prevent protocol collisions.


## Rationale

### Commitments with a set of public keys, not a single key

The protocol was designed to support commitments to multiple public keys in
order to be usable with non-P2(W)PK outputs. For instance, with Lightning 
network all outputs in the commitment transaction are non-P2WPK, so all existing
key tweaking schemes are not usable within LN structure.

### Use of HMAC insead of simple hash

Reason: prevention of length-extension attacks

As this protocol aims to be a generic scheme, the message `msg` can be of any
length. If we would just use a simple hash (e.g. SHA256), users of `LNPBP-1`
could **potentially** be vulnerable to length-extension attacks, if they are not
careful. To be on the safe side, we use HMAC-SHA256, which is resistant to
length-extension attacks, but computationally more expensive. However, this
protocol aims to be used in client-side validation applications primarily and
should therefore run many orders of magnitude less often then complete
validatation of all public blockchain data. The computational overhead of HMAC
on a client node is therefore considered negligible, for the targeted use cases.

### Public key serialization to 64 byte uncompressed form

Reason: HMAC needs a byte array as input

HMAC requires a byte array as input for the `key` argument to authenticate a
message. This `key` is not intended to be an EC key, it can be anything. Its
purpose is to add entropy to the resulting hash value to counter length attacks
on the underlying message.

We use HMAC's `key` argument for two purposes:
1. Commit the message `msg` to a specific public key `S`.
2. As entropy for the security of HMAC-SHA256 against length extension attacks.

For the serialization of the public key `S`, we rely on the *de facto* standard
format for uncompressed public keys in Bitcoin, which is followed by libraries
like [rust-secp256k1](https://docs.rs/secp256k1/0.20.1/src/secp256k1/key.rs.html#290-306). 
However, this results in a 65 byte array with the first byte being the prefix
having the value `0x04`, denoting an uncompressed public key. However, the first
byte doesn't add any entropy and a `key` larger than 64 byte causes HMAC-SH256
to do an additional round of hashing. Therefore, we use `rust-secp256k1`'
s `key.serialize_uncompressed()` function, but strip the first byte from the
resulting value, so we end up with a 64 byte array of:

- 32 bytes representing the x coordinate in big-endian order,
- followed by 32 bytes representing the y coordinate in big-endian order.

### Use of protocol tags

Reason: prevention of cross-protocol collision attacks

The use of protocol-specific, double tagged hashes was originally proposed by 
Peter Wuille in [4] and [5] in order to prevent potential replay-attacks for
interpreting messages under different protocols. The choice of a duplicate 
SHA256 prefix hash was made according to Peter Wuille because it is not yet
used in any existing bitcoin protocol, which increases compatibility and reduces
chances of collisions with existing protocols.

### Protocol failures

The protocol may fail during some of the **commitment** procedure steps:

* when the *tweaking factor* `f` exceeds the order `n` of the generator
  point `G` for the selected elliptic curve.
* when the multiplication of the Secp256k1 generator point `G` on the *tweaking 
  factor* `f` results in `F` being equal to the point at infinity.
* when the summation of the members of public key set `P` at any stage, or the 
  addition of the point `F` with the original public key `Po`, results in
  the point at infinity.

The probabilities of these failures are infinitesimal; for instance the
probability of the SHA256 hash value of a random message exceeding `G` order `n` 
is `(2^256 - n) / n`, which is many orders of magnitude less than the 
probability of a CPU failure. The probability of the second or third failure is 
even lower, since the point at infinity may be obtained only if `F` is equal to 
`-G` or `-P`, i.e. the probability of private key collision, equal to the
inverse of Secp256k1 curve generator point order `n`. The only reason why this
kind of failure may happen is when the original public key set was forged in a
way that some of its keys are equivalent to the negation of other keys.

These cases may be ignored by a protocol user -- or, alternatively, in case of
the protocol failure the user may change `P`'s value(s) and re-run the protocol.

Protocol failures during the verification procedure may happen only during its
repetition of the original commitment. This means that the original commitment
is invalid, since it was not possible to create a commitment with the given
original data. Thus, such failure will simply indicate a negative result of the
verification procedure.

### Choice of elliptic curve generator point order `n` over field order `p`

While it is possible to ignore elliptic curve overflow over its order `n` during
public key addition, since it does not provide a security risk for the 
commitment, it was chosen to stick to this scheme because of the following:

* Current implementation of Secp256k1 library (libsecp256k1) fails on overflow
  during key tweaking procedure. Since this library is widely used in the
  Bitcoin ecosystem (and Bitcoin Core), it is desirable to maintain LNPBP-1
  compatible with this functionality.
* Probability of an overflow is still infinissimal, being comparable to
  probability of `3.7*10^-66`, for a tweaking factor not fitting into the
  elliptic curve field order `p`.

### No nonce

In certain circumstances a simple hash based commitment might be vulnerable to
brute force vocabulary attacks, if the syntax and semantics of the invoking
protocol are known to the attacker. This is usually countered with adding
additional entropy (e.g. a nonce) to each hash. In our case the public key `S`
already provides enough entropy, which - when added via HMAC-SHA256 to the
whole `msg` – sufficiently counters such vocabulary attacks, preventing an
attacker from successfully guessing the original message, even for short and 
standard messages.


## Reference implementation

```rust
use std::collections::BTreeSet;

use bitcoin::hashes::{sha256, Hash, HashEngine, Hmac, HmacEngine};
use bitcoin::secp256k1;

lazy_static! {
    /// Global Secp256k1 context object
    pub static ref SECP256K1: secp256k1::Secp256k1<bitcoin::secp256k1::All> = 
      secp256k1::Secp256k1::new();
}

lazy_static! {
    /// Single SHA256 hash of "LNPBP1" string according to LNPBP-1 acting as a
    /// prefix to the message in computing tweaking factor
    pub static ref LNPBP1_HASHED_TAG: [u8; 32] = {
        sha256::Hash::hash(b"LNPBP1").into_inner()
    };
}

/// Deterministically-organized set of all public keys used by this mod
/// internally. 
type Keyset = BTreeSet<secp256k1::PublicKey>;

/// Errors that may happen during LNPBP-1 commitment procedure or because of
/// incorrect arguments provided to [`commit()`] function.
#[derive(Clone, Copy, PartialEq, Eq, Debug, Display, Error, From)]
#[display(doc_comments)]
pub enum Error {
    /// Keyset must include target public key, but no target key found it
    /// the provided set.
    NotKeysetMember,

    /// Elliptic curve point addition resulted in point in infinity; you
    /// must select different source public keys
    SumInfiniteResult,

    /// LNPBP-1 commitment either is outside of Secp256k1 order `n` (this event
    /// has negligible probability <~2^-64), or, when added to the provided
    /// keyset, results in point at infinity. You may try with a different
    /// source message or public keys.
    InvalidTweak,
}

/// Function performs commitment procedure according to LNPBP-1.
pub fn commit(
    // We automatically reduce set to unique keys by using `BTreeSet` in the 
    // underlying `Keyset` data type
    keyset: &mut Keyset,
    target_pubkey: &mut secp256k1::PublicKey,
    // We take a hashed version of the protocol tag for computation efficiency
    // so it can be used in multiple commitments without hash re-computing
    protocol_tag: &sha256::Hash,
    message: &impl AsRef<[u8]>,
) -> Result<[u8; 32], Error> {
    if !keyset.remove(target_pubkey) {
        return Err(Error::NotKeysetMember);
    }

    let pubkey_sum = keyset
        .iter()
        .try_fold(target_pubkey.clone(), |sum, pubkey| sum.combine(pubkey))
        .map_err(|_| Error::SumInfiniteResult)?;

    let mut hmac_engine =
        HmacEngine::<sha256::Hash>::new(&pubkey_sum.serialize_uncompressed());

    hmac_engine.input(&LNPBP1_HASHED_TAG[..]);
    hmac_engine.input(&protocol_tag[..]);
    hmac_engine.input(&sha256::Hash::hash(message.as_ref()));

    // Producing and storing tweaking factor in container
    let hmac = Hmac::from_engine(hmac_engine);
    let tweaking_factor = *hmac.as_inner();

    // Applying tweaking factor to public key
    target_pubkey
        .add_exp_assign(&SECP256K1, &tweaking_factor[..])
        .map_err(|_| Error::InvalidTweak)?;

    keyset.insert(target_pubkey.clone());

    // Returning tweaked public key
    Ok(tweaking_factor)
}

/// Function verifies commitment created according to LNPBP-1.
pub fn verify(
    verified_pubkey: secp256k1::PublicKey,
    original_keyset: &Keyset,
    mut target_pubkey: secp256k1::PublicKey,
    protocol_tag: &sha256::Hash,
    message: &impl AsRef<[u8]>,
) -> bool {
    match commit(
        &mut original_keyset.clone(),
        &mut target_pubkey,
        protocol_tag,
        message,
    ) {
        // If the commitment function fails, it means that it was not able to
        // commit with the provided data, meaning that the commitment was not
        // created. Thus, we return that verification have not passed, and not
        // a error.
        Err(_) => return false,

        // Verification succeeds if the commitment procedure produces public key
        // equivalent to the verified one
        Ok(_) => target_pubkey == verified_pubkey,
    }
}
```

## Acknowledgements

Authors would like to thank:
* Alekos Filini for his initial work on the commitment schemes as a part of 
  early RGB effort [6];
* ZmnSCPxj for bringing attention to possible Taproot-compatibility issues [7];
* Peter Wuille for a proposal on the tagged hashes, preventing reply-type of 
  attacks [5];
* Authors of Sidechains whitepaper for paying attention to the potential 
  length-extension attacks and the introduction of HMAC-based commitments to the 
  original public key [3];
* Dr Christian Decker for pointing out on Lightning Network incompatibility with
  all existing cryptographic commitment schemes.


## References

1. Ilja Gerhardt, Timo Hanke. Homomorphic Payment Addresses and the
   Pay-to-Contract Protocol. arXiv:1212.3257 \[cs.CR\] 
   <https://arxiv.org/pdf/1212.3257.pdf>
2. [Eternity Wall's "sign-to-contract" article](https://blog.eternitywall.com/2018/04/13/sign-to-contract/)
3. Adam Back, Matt Corallo, Luke Dashjr, et al. Enabling Blockchain Innovations 
   with Pegged Sidechains (commit5620e43). Appenxix A. 
   <https://blockstream.com/sidechains.pdf>.
4. Pieter Wuille. Schnorr Signatures for secp256k1.
   <https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki>
5. Pieter Wuille. Taproot: SegWit version 1 spending rules.
   <https://github.com/bitcoin/bips/blob/master/bip-0341.mediawiki>
6. RGB Protocol Specification, version 0.4. "Commitment Scheme" section.
   <https://github.com/rgb-org/spec/blob/old-master/01-rgb.md#commitment-scheme>
7. <https://github.com/rgb-org/spec/issues/61>


## License

This document is licensed under the Creative Commons CC0 1.0 Universal license.


## Appendix A. Test vectors

All tests done with protocol-specific `tag` value equal to `ProtoTag`. Values
for public keys are given in standard compressed pre-Shorr's Bitcoin encoding 
format; values for tweaking factors are given in little-endian byte order.

### 1. Correct test vectors

#### 1.1. Zero-length message

1) Single public key #1
  - Original public key: 
    `03ab1ac1872a38a2f196bed5a6047f0da2c8130fe8de49fc4d5dfb201f7611d8e2`
  - Tweaked public key with the commitment:
    `025d69da2890f85928cb492545a13bd6782168b39d52e69fadd1d3fcb3b1bf9268`
  - Tweaking factor value:
    `9ff4c975950ec102b5eb39df2f976948b2c1a6e3f92ef5bf5af0e1241380dbcf`
2) Single public key #2
  - Original public key: 
    `039729247032c0dfcf45b4841fcd72f6e9a2422631fc3466cf863e87154754dd40`
  - Tweaked public key with the commitment:
    `032fdf6c4023453b869294ddd28684f98fcaca604c2cd734c8dd64b8520547b0b4`
  - Tweaking factor value:
    `11db141cfe0143f60e9e9f9db478630033fc65eb4f682905e9044c87869459a5`
3) Set of five public keys
  - Original public key: 
    `02383b24fbea14253ac37b0d421263b716a34192516ea0837021a40b5966a06f5e`
  - Key set: 
    * `02383b24fbea14253ac37b0d421263b716a34192516ea0837021a40b5966a06f5e`
    * `025b178dfaa49e959033cc2ba8b06d78b8b9242496329a574eb8e2b4fad4f88b6f`
    * `03ec8b1cf223dc3cd8eb6d7c5fb11735e983c234b69271a3decad8bbfb2b997994`
    * `021ce48f4b53257be01ccb237986c1b9677a9e698fb962b108d6b2fbdc836727d8`
    * `0388a0fc8d3ba29a93ad07dbad37a6d4b87f2e2672b15d331d1f6bf4f2c9119ffe`
  - Tweaked public key with the commitment:
    `03c153beef57c268ee9a2a68940f2aa7b052ce14c676a27cfe5010c53b41476238`
  - Tweaking factor value:
    `a18417ae90cf36a45311ccc3a911a8ebb1b7afa02c6d79d1d1bd08b2abf67e94`
4). Set of same five public keys, using different original key from the set
  - Original public key: 
    `025b178dfaa49e959033cc2ba8b06d78b8b9242496329a574eb8e2b4fad4f88b6f`
  - Key set: 
    * `02383b24fbea14253ac37b0d421263b716a34192516ea0837021a40b5966a06f5e`
    * `025b178dfaa49e959033cc2ba8b06d78b8b9242496329a574eb8e2b4fad4f88b6f`
    * `03ec8b1cf223dc3cd8eb6d7c5fb11735e983c234b69271a3decad8bbfb2b997994`
    * `021ce48f4b53257be01ccb237986c1b9677a9e698fb962b108d6b2fbdc836727d8`
    * `0388a0fc8d3ba29a93ad07dbad37a6d4b87f2e2672b15d331d1f6bf4f2c9119ffe`
  - Tweaked public key with the commitment:
    `03a224242255c9a024d4e2723c17faa09082b60bf91cea23ce558c9cff3a9627bf`
  - Tweaking factor value:
    `a18417ae90cf36a45311ccc3a911a8ebb1b7afa02c6d79d1d1bd08b2abf67e94`

#### 1.2. Message consisting of a single zero byte

1) Single public key #1
    - Original public key: 
      `032564fe9b5beef82d3703a607253f31ef8ea1b365772df434226aee642651b3fa`
    - Tweaked public key with the commitment:
      `0285f7e0a8cdd801e5fbf84602e84de46a036ba47230b2c37f7767a496aeb4e4c5`
    - Tweaking factor value:
      `5639647143cb9dc78aa5d251694fcc053f3887cf27b13750f72a42ef04f7bde1`
2) Single public key #2
    - Original public key: 
      `0289637f97580a796e050791ad5a2f27af1803645d95df021a3c2d82eb8c2ca7ff`
    - Tweaked public key with the commitment:
      `03fcd2e4c31622fcf9fef43e70dabf1daf8abae5685b15125ba6a0e444783c5f0e`
    - Tweaking factor value:
      `7551544f39a2c3a4d65c34e5915702a825ccbbb914ac581389cbbd98869b4e48`
3) Set of five public keys
    - Original public key: 
      `03ff3d6136ffac5b0cbfc6c5c0c30dc01a7ea3d56c20bd3103b178e3d3ae180068`
    - Key set: 
      * `03ff3d6136ffac5b0cbfc6c5c0c30dc01a7ea3d56c20bd3103b178e3d3ae180068`
      * `02308138e71be25e092fdc9da03d5357421bc7280356a1381a6186d63a0ca8dd7f`
      * `03575fc4e82a6deb65d1e5750c85b6862f6ec009281992e206c0dcc568866a3fb1`
      * `0271efa4e26a4179e112860b88fc98658a4bdbc59c7ab6d4f8057c35330c7a89ee`
      * `0289637f97580a796e050791ad5a2f27af1803645d95df021a3c2d82eb8c2ca7ff`
    - Tweaked public key with the commitment:
      `0289d1313a940f7b668804e223662edce2a7138914894607cd4bf641cc584936f3`
    - Tweaking factor value:
      `87a5728772e0d14c9938c50ab29b215d5a0d9f59be7b40d16cc4bcac22e027b1`

#### 1.3. Message of text string `test`

1) Single public key #1
    - Original public key: 
      `0271efa4e26a4179e112860b88fc98658a4bdbc59c7ab6d4f8057c35330c7a89ee`
    - Tweaked public key with the commitment:
      `02605b2400618ca83f563e997da456c7ae99df9b38a7939ead5bc8e5b8b29f5d45`
    - Tweaking factor value:
      `7090ad6b1c6093e025c3b2f1607f9aea65449139a08ee773c61990e9b6e966d3`
2) Single public key #2
    - Original public key: 
      `039729247032c0dfcf45b4841fcd72f6e9a2422631fc3466cf863e87154754dd40`
    - Tweaked public key with the commitment:
      `032bf20cd8539c2f3154fbae01e64ea3a492bb2431080c86c3f942571f9635ece7`
    - Tweaking factor value:
      `214570a96bf958124eea266593fd9daed3ee357283b4f89613f99a5d8ac8910a`
3) Set of five public keys
    - Original public key: 
      `03f72a42169a0475c4a342f8da97a1c0bce830183efecd0a3d81637b05d7c0d81a`
    - Key set: 
      * `03f72a42169a0475c4a342f8da97a1c0bce830183efecd0a3d81637b05d7c0d81a`
      * `02383b24fbea14253ac37b0d421263b716a34192516ea0837021a40b5966a06f5e`
      * `025b178dfaa49e959033cc2ba8b06d78b8b9242496329a574eb8e2b4fad4f88b6f`
      * `03ec8b1cf223dc3cd8eb6d7c5fb11735e983c234b69271a3decad8bbfb2b997994`
      * `03f0d2dd91c4bcb630616ea9e3b2e95ec7f6f431d81bd627b62d04ac81b91af8c7`
    - Tweaked public key with the commitment:
      `02da1eea3c29872e9d770efe66bfde4ad2b361f0644e81d1b4d95338eb75b813f1`
    - Tweaking factor value:
      `63ea2d88f3b3969573ef530132989a9281cb499d6bfda4bfc0ade2cbd7bdf26e`

#### 1.4. Binary messsage, hex encoding (little-endian byte order)

Original message for the all cases in this section: `[0xde, 0xad, 0xbe, 0xef]`

1) Single public key #1
    - Original public key: 
      `0352045bcc58e07124a375ea004b3508ac80e625da2106c74f5cb023498de0545f`
    - Tweaked public key with the commitment:
      `0357f2619c2805794ef65ab7ea7a349f4c1be4cc3f576584f8270f06e830f33e36`
    - Tweaking factor value:
      `14703d20ec36407889e5d7546d59edbfac4e69f211759a1bd783aa65ee1ae36c`
2) Single public key #2
    - Original public key: 
      `02a153dfe913310b0949de7976146349b95a398cb0de1047290b0f975c172ad712`
    - Tweaked public key with the commitment:
      `0388bcce7da0bc2edd2ff553134c7ae109232f30bda347b39adca6d0d379a86315`
    - Tweaking factor value:
      `627573dc2a7a57e5fe83f415d5f9d0e9ee78e51fd7990e926f09e9b8fe6a12b3`
3) Set of five public keys
    - Original public key: 
      `03a9c44838c0ac7417497f770ebd013c91ac715665ec01e740be0e14f44cab2474`
    - Key set: 
      * `03a9c44838c0ac7417497f770ebd013c91ac715665ec01e740be0e14f44cab2474`
      * `03ad42e3bd69e30d32d088173e02b9d1cd00e4f7d945aad5c1a6c9439fdc8c5e80`
      * `03713e80a43b19d6f7b46ec5a474e86c8f5769f85f4fcb9a0be76d095b1e2b7981`
      * `025d9e055d7e7a85f097e981779c6e1c40d74b0563e631128c06623609b99a8f87`
      * `0323e518565f25038f16fdf7686ed4dd9a59b02ef95d2d7aa5be948f38701376b7`
    - Tweaked public key with the commitment:
      `02d739f0fdd7bc395482c52e1ef1547a3c6fc6e2f1393430e74c55624f26023bd7`
    - Tweaking factor value:
      `d5218633603181303d06320365fc84d06e0c2bb36c0989ee678a57b799f457a7`

#### 1.5. Keyset changes

Commitment creation and validation filters repeated keys and does not depend
on the key order (since elliptic curve addition is commutative)

1) Set of five public keys containing duplicated keys
    - Message (binary string, little-endian byte order):
      `[0x00, 0xde, 0xad, 0xbe, 0xef]`
    - Original public key: 
      `025d9e055d7e7a85f097e981779c6e1c40d74b0563e631128c06623609b99a8f87`
    - Key set: 
      * `03a9c44838c0ac7417497f770ebd013c91ac715665ec01e740be0e14f44cab2474`
      * `03ad42e3bd69e30d32d088173e02b9d1cd00e4f7d945aad5c1a6c9439fdc8c5e80`
      * `03ad42e3bd69e30d32d088173e02b9d1cd00e4f7d945aad5c1a6c9439fdc8c5e80`
      * `03713e80a43b19d6f7b46ec5a474e86c8f5769f85f4fcb9a0be76d095b1e2b7981`
      * `025d9e055d7e7a85f097e981779c6e1c40d74b0563e631128c06623609b99a8f87`
      * `0323e518565f25038f16fdf7686ed4dd9a59b02ef95d2d7aa5be948f38701376b7`
      * `025d9e055d7e7a85f097e981779c6e1c40d74b0563e631128c06623609b99a8f87`
    - Tweaked public key with the commitment:
      `027f07015596c7a3af8a1da9e4fe1de0695278f94278ce01534b7ac7a530b43399`
    - Tweaking factor value:
      `bc47cf269e70e5e654f3079f7316ddd988c529bf7d8c0efb0ec0759719afaeaa`
2) Set of five public keys in changed order
    - Message (binary string, little-endian byte order):
      `[0x00, 0xde, 0xad, 0xbe, 0xef]`
    - Original public key: 
      `025d9e055d7e7a85f097e981779c6e1c40d74b0563e631128c06623609b99a8f87`
    - Key set: 
      * `03713e80a43b19d6f7b46ec5a474e86c8f5769f85f4fcb9a0be76d095b1e2b7981`
      * `03a9c44838c0ac7417497f770ebd013c91ac715665ec01e740be0e14f44cab2474`
      * `025d9e055d7e7a85f097e981779c6e1c40d74b0563e631128c06623609b99a8f87`
      * `0323e518565f25038f16fdf7686ed4dd9a59b02ef95d2d7aa5be948f38701376b7`
      * `03ad42e3bd69e30d32d088173e02b9d1cd00e4f7d945aad5c1a6c9439fdc8c5e80`
    - Tweaked public key with the commitment:
      `027f07015596c7a3af8a1da9e4fe1de0695278f94278ce01534b7ac7a530b43399`
    - Tweaking factor value:
      `bc47cf269e70e5e654f3079f7316ddd988c529bf7d8c0efb0ec0759719afaeaa`

### 2. Invalid test vectors

All these cases are cases for validation procedure, which must fail.

1) Case #1: commitment key created with a different original public key
    - Message: zero-length
    - Original public key: 
      `03ab1ac1872a38a2f196bed5a6047f0da2c8130fe8de49fc4d5dfb201f7611d8e2`
    - Tweaked public key with the commitment:
      `02a8e7b5f006e3c96eb1e336d40a6956dd9c4889dbfb4542b50da0c90cd2ab64fd`

2) Case #2: original key and commitment are valid, but the message was different
    - Message: `test*`
    - Original public key: 
      `032564fe9b5beef82d3703a607253f31ef8ea1b365772df434226aee642651b3fa`
    - Tweaked public key with the commitment:
      `0240c2f382fc5335879c3607479c491dbd9bfb47d32c375f7d99e6d210a91f8780`

3) Case #3: commitment was created with correct message and original public key,
   but using different protocol tag
    - Message (binary string, little-endian byte order):
      `[0xde, 0xad, 0xbe, 0xef, 0x00]`
    - Original public key: 
      `029a541ac6af794615935c34d088edc824c4433a83bdb5a781030c370111cf5b3a`
    - Tweaked public key with the commitment:
      `0304d89459380b9d8ff2ebaaf2e20f47ce92dcf0b9dbfde9dbe866513a7819b79c`

4) Case #4: one of original public keys is absent
    - Message: 
      `test`
    - Original public key: 
      `03f72a42169a0475c4a342f8da97a1c0bce830183efecd0a3d81637b05d7c0d81a`
    - Key set: 
      * `03f72a42169a0475c4a342f8da97a1c0bce830183efecd0a3d81637b05d7c0d81a`
      * `02383b24fbea14253ac37b0d421263b716a34192516ea0837021a40b5966a06f5e`
      * `03ec8b1cf223dc3cd8eb6d7c5fb11735e983c234b69271a3decad8bbfb2b997994`
      * `03f0d2dd91c4bcb630616ea9e3b2e95ec7f6f431d81bd627b62d04ac81b91af8c7`
    - Tweaked public key with the commitment:
      `02da1eea3c29872e9d770efe66bfde4ad2b361f0644e81d1b4d95338eb75b813f1`


### 3. Edge cases: protocol failures

Keyset constructed of a key and it's own negation. 

- Expected result: 
  must fail commitment procedure with error indicating that the operation 
  resulted at the point-at-infinity.
- Message: 
  `test`
- Original public key: 
  `0218845781f631c48f1c9709e23092067d06837f30aa0cd0544ac887fe91ddd166`
- Key set: 
  * `0218845781f631c48f1c9709e23092067d06837f30aa0cd0544ac887fe91ddd166`
  * `0318845781f631c48f1c9709e23092067d06837f30aa0cd0544ac887fe91ddd166`
