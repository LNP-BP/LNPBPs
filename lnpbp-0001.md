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
Comments-URI: https://github.com/LNP-BP/lnpbps/issues/3
Status: Proposal
Type: Standards Track
Created: 2019-09-23
License: CC0-1.0
```

## Abstract

The work proposes a standartization for cryptographic commitments that utilize
elleptic curve (EC) homomorphic properties and allow to commit to an arbitrary
data using EC public keys or a set of EC public keys in a deterministic and safe
way.

## Background

Cryptographic commitments represent a way to commit to some message without
revealing it. The procedure consists of two phases, **commit** and **reveal**.
In the *commit* phase a party (**committer**) willing to prove its knowledge of
some message computes a *cryptographic hash function* over that message,
producing a message **digest**, which can be provided to other party(ies). In
the *reveal* phase *committer* reveals the actual message, and each party
accessing it may check that it's hash is equal to the originally provided
*digest*.

Key tweaking is a procedure for creation of cryptographic commitment to some 
**message** using elliptic curve properties. The procedure uses descrete log
problem (DLP) as a prove of existance & knowledge of certain information about
the message by some party (Alice) without exposing the original message. This is
done by adding to a public key, for which Alice knows corresponding private key,
a hash of the message multiplied on the generator point `G` of the elliptic
curve. This produces a **tweaked** public key, containing the commitment. At a
later time Alice may prove her past knowledge of the original message (at the
time when the commitment was created) by providing a signature corresponding to
the original public key and the message itself.

Main advantage of the public key tweak procedure is the fact that a tweaked key,
or a corresponding signature, can't be distinguished from any other public keys
or signatures; this property allows to hide the actual commitment in such a way
that it can be only known to those parties which have knowledge of a secrets:
original public and/or key pair **and** a message.

This type of commitment was originally proposed as a part of "pay to contract"
concept by Ilja Gerhardt and Timo Hanke in [1] and later used by Eternity Wall
[2] for the same purpose. However, these proposals were arguably vulnerable to
length-extension attacks and, more importangly, was not applicable to scenatious
when multiple public keys are uses (for instance, multi-signature bitcoin
transaction outputs). These problems were fixed as a part of sidechain-design
effort by Blockstream [3], which proposed to utilize HMAC function and
introduced concept of nonce.

Here we propose a standartization of the algorithm based on the original
Eternity Wall and Blockstream work, enhanced with Pieter Wuille Tagged Hashes
procedure, coming from a specification on Schnorr signatures in Bitcoin [4],
also used in Taproot proposal [5]. This procedure prevents cross-protocol
collisions, such that original message byte sequence can't be reinterpreted
under other protocol.


## Motivation

Publication of cryptographic commitments to Bitcoin blockchain is a widely used
mechanism, allowing timestamping of the commitment: it can be used to prove the
fact that some information was known before certain period in time without
revealing the actual information. Use of elliptic curve homomorphic properties
allows to perform such commitments without increasing the size of the
transaction, by leveraging existing transaction outputs and not polluting
blockchain space with excessive OP_RETURN's. However, as of today, there is no
single standard for such commitments, while different practices, used for the
purpose (see [1, 2, 3]) contain multiple collision risks, such as possibility of
the length-extension attacks, cross-protocol reply attacks or can't be applied
in situations where multiple public keys are used (multisignature or custom
bitcoin scripts). This standard combines existing best practices into a single
algorithm, that avoids all of those issues.


## Specification

For a given message `msg`, a set of unique public keys `P := {P1, P2, ..., Pn}`,
`n > 0`, with some selected original public key `Po` from this set (`Po ∈ S`) 
**commit procedure** runs as follows:

1. Compute a sum `S` of all public keys in set `P`; fail the protocol if an 
   overflow over elliptic curve generator point order happens during the 
   procedure.
2. Construct a byte string `lnpbp1_msg`, composed of the original message
   prefixed with a single SHA256 hash of `LNPBP1` string and a single SHA256 
   hash of protocol-specific tag: 
   `lnpbp1_msg = SHA256("LNPBP1") || SHA256(<protocol-specific-tag>) || msg`
3. Compute HMAC-SHA256 of the `lnbp1_msg` using sum of public keys `S`. The 
   resulting value is named **tweaking factor** `f`: 
   `f = HMAC_SHA256(lnpbp1_msg, S)`
4. Make sure that the tweaking factor is less than order `n` of a generator 
   point of the used elliptic curve, such that no overflow can happen when it is 
   added to the original public key. If the order is exceeded, fail the protocol
   indicating the reason of failure.
5. Multiply the tweaking factor on the used elliptic curve generator point `G`: 
   `F = G * f`
6. Check that the result of 5 is not equal to the point-at-infinity; otherwise 
   fail the protocol, indicating the reason of failure, such that the protocol 
   may be run with another initial public key `P'` value.
7. Add two elliptic curve points: the original public key `Po` and 
   point `F`, derived from the tweaking-factor. This will give resulting tweaked 
   public key `T`: `T = Po + F`. Check that the result is not equal to the 
   point-at-infinity of the elliptic curve and fail the protocol otherwise, 
   indicating the reason of failure, such that the protocol may be run with 
   another initial public key set.

The final formula for the commitment is:
`T = Po + G * HMAC_SHA256(SHA256("LNPBP1")||SHA256(<protocol-specific-tag>)||msg, S)`

**Reveal procedure** for the commitment (i.e. tweaked public key `T`) can be 
performed with the provision of the original public key `P` and message `msg` 
(assuming that the verifying party is aware of the protocol-specific tag
and `LNPBP1` tag) and runs as follows:

1. Make sure that the provided tweaked public key `T` lies on the elliptic curve 
   and is not equal to the point at infinity.
2. Compute 
   `T' = Po + G * HMAC_SHA256(SHA256("LNPBP1")||SHA256(<protocol-specific-tag>)||msg, S)` 
   repeating the *commitment procedure* according to the rules above.
3. Make sure that `T' = T` and report verification success; otherwise report 
   verification failure.

Thus, **proof data** required for the commitment verification constist of:

1. Original message `msg`
2. Tweaked public key value `T`
3. Original set of public keys `P` and a key `Po` from that set.

The used protocol tag `tag` must be known to all parties participating in the 
protocol.


## Compatibility

The proposed procedure should be compatible with previously-created
pay-to-contract-style commitments based on SHA256 hashes under an assumption of
SHA256 collision resistance. Utilization of duplicated protocol tag hash prefix
guarantees randomness in the first 64 bytes of the resulting tweaking string
`lnpbp1_msg`, reducing probability for these bytes to be interpreted as a
correct message under one of the previous standards.

The tweaked procedure may result in public key that may, or may not have its y
coordinate being a quadratic residue (in terms of BIP-340 [4]). This may present
a compatibility issue for using this scheme in Taproot/Schnorr-enabled outputs
and protocols. Nevertheless, this issue may be mitigated by running the
procedure  second time and replacing the original public key with its own
negation if the resulted tweaked version  was not square.

Proposal relies on a tagged hash prefix similar to the one used in BIP-340, [4],
which helps to prevent protocol collisions.


## Rationale

### Use of HMAC: prevention of length extension attacks and multiple commitments

### Use of protocol tags: prevention of inter-protocol collision attacks

The use of duplicated protocol-specific tag hash is originally proposed by Peter
Wuille in [4] and [5] in order to prevent potential reply-attacks for
interpreting message under different protocols. The choice of duplicate single
SHA256 hash is made due to the fact that according to Peter Wuille it is not yet
used in any existing bitcoin protocol, which increases compatibility and reduces
chances of collisions with existing protocols.

### Protocol failures

The protocol may fail during some of **commitment** procedure steps:
* when the *tweaking factor* `f` exceeds order `n` of the generator point `G`
  for the selected elliptic curve.
* when the multiplication of Secp256k1 generator point `G` on the *tweaking 
  factor* `f` results in `F` equal to the point at infinity
* when the summation of the members of public key set `P` at any stage, or the 
  addition of the point `F` and the original public key `P` results in
  the point at infinity.

The probabilities of these failures are infinitesimal; for instance the
probability of SHA256 hash value of a random message exceeding `G` order `n` is
`(2^256 - n) / n`, which is many orders of magnitude is less than a probability
of CPU failure. The probability of the second and third failure is even lower,
since the point at infinity may be obtained only if `F` will be equal to `-G` or
`-P`, i.e. the probability of private key collision, equal to the inverse of
Secp256k1 curve generator point order `n`. The only reason why this kind of 
failure may happen is when the original public key set was forged in a way that
some of its key are equivalent to the negation of other keys.

These cases may be ignored by a protocol user -- or, alternatively, in case of
the protocol failure the user may change `P` value and re-run the protocol.

Protocol falure during the verification procedure may happen only during its
repetition of the original commitment. This will mean that the original 
commitment is invalid, since it was not possible to create any commitment with
the given original data. Thus, such failure will simpli indicate negative result
of the verification procedure.

### Choise of elliptic curve generator point order `n` over field order `p`

While it is possible to ignore elliptic curve overflow over its order `n` during
public key addition, since it does not provide a security risk for the 
commitment, it was chosen to stick to this scheme because of the following:
* Current implementation of Secp256k1 library (libsecp256k1) fails on overflow
  during key tweaking procedure. Since this library is widely used in bitcoin
  ecosystem (and Bitcoin Core) it is desirable to maintain LNPBP-1 compativle
  with its functionality
* Probability of the overflow is still infinissimal, being comparable to
  probability of `3.7*10^-66` for a tweaking factor not fitting into the
  elliptic curve field order `p`.

### No nonce

Nonce is not required since the original public key, hidden in the resulting
tweaked key, presents sufficient entropy for an attacker to be not able to guess
the original message even for short and standard messages.


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
/// internally
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
    keyset: &mut Keyset,
    target_pubkey: &mut secp256k1::PublicKey,
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
        HmacEngine::<sha256::Hash>::new(&pubkey_sum.serialize());

    hmac_engine.input(&LNPBP1_HASHED_TAG[..]);
    hmac_engine.input(&protocol_tag[..]);
    hmac_engine.input(message.as_ref());

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


## References

1. Ilja Gerhardt, Timo Hanke. Homomorphic Payment Addresses and the
   Pay-to-Contract Protocol. arXiv:1212.3257 \[cs.CR\] 
   <https://arxiv.org/pdf/1212.3257.pdf>
2. [Eternity Wall's "sign-to-contract" article](https://blog.eternitywall.com/2018/04/13/sign-to-contract/)
3. Adam Back, Matt Corallo, Luke Dashjr, et al. Enabling Blockchain Innovations 
   with Pegged Sidechains (commit5620e43). Appenxix A. 
   <https://blockstream.com/sidechains.pdf>.
4. Pieter Wuille. Schnorr Signatures for secp256k1.
   <https://github.com/sipa/bips/blob/master/bip-340.mediawiki>
5. Pieter Wuille. Taproot: SegWit version 1 spending rules.
   <https://github.com/sipa/bips/blob/master/bip-341.mediawiki>
6. RGB Protocol Specification, version 0.4. "Commitment Scheme" section.
   <https://github.com/rgb-org/spec/blob/old-master/01-rgb.md#commitment-scheme>
7. <https://github.com/rgb-org/spec/issues/61>


## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.


## Test vectors

TBD
