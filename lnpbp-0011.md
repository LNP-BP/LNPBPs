```
LNPBP: 0011
Layer: Client-validated graphs (4)
Vertical: Smart contracts
Title: RGB: Client-validated confidential smart contracts using bitcoin
       transaction graphs for Bitcoin and Lightning Network
Authors: Dr Maxim Orlovsky <orlovsky@protonmail.ch>,
         Giacomo Zucco,
         Alekos Filini,
         Peter Todd <pete@petertodd.org>,
         Federico Tenga,
         Marco Amadori,
         Martino Salvetti,
         Nicola Busanello
Comments-URI:
Status: Proposal
Type: Informational
Created: 2020-05-12
License: CC0-1.0
Related standards: LNPBP1-4, 6-10, 12-14, 20-29, 33-35, 37
```

## Abstract

RGB is a smart contract system, able to work on top of the Lightning Network
and designed with confidentiality and scalability in mind. It is maintained
by [LNP/BP Standards Association](https://github.com/LNP-BP).

What is possible with RGB:
* Issue digital fungible assets, like stock, bonds and other forms of securities;
* create different forms of collectibles (non-fungible assets);
* create and manage sovereign/decentralized identities and reputation systems;
* create and maintain provably-unique history of some events that may be used
  for audit with well-controlled partial disclosure of the data;
* design and run other forms of arbitrary-complex smart contracts

## Background

## Motivation

## Design

As a smart contract system RGB is quite different from previous approaches, both
Bitcoin-based (Colored coins, Counterparty, OMNI) and non-bitcoin (Ethereum, EOS
and others):

* RGB separates concept of smart contract **issuer**, **state owners** and
  **state evolution**
* RGB keeps the smart contract code and data offchain
* RGB uses blockchain as a state commitment layer and Bitcoin script as an
  ownership control system; while smart contract evolution is defined by
  off-chain **schema** and Turing-complete scripting system using *Simplicity
  language*

More about these concepts can be read in
[this presentation](https://github.com/LNP-BP/FAQ/blob/master/Presentation%20slides/RGB%20%26%20Spectrum%20explanation%20for%20business.pdf).

Briefly, RGB smart contracts operate with **client-side validation** paradigm,
meaning that all the data is kept outside of the bitcoin transactions, i.e.
bitcoin  blockchain or lightning channel state. This allows the system to
operate on top of Lightning Network without any changes to the LN protocols and
also gives a foundation for a high level of protocol scalability and privacy.

As a security mechanism RGB uses **single-use seals** defined over bitcoin
transaction outputs, which provides ability for any party having smart contract
state history to verify it's uniquiness. In other words, RGB leverages Bitcoin
script for its security model and definition of the **ownership** and **access
rights**.

Each RGB smart contract is represented by some **genesis state**, created by
**smart contract issuer** (or, put simply, issuer) and a directed acyclic graph
(DAG) of **state transitions** kept in form of *client-validated data* (i.e.
this data is not stored on blockchain or within LN transactions/channel state).
The state is **assigned** to unspent bitcoin transaction outputs, which defines
them as *single-use seals*. The party that is able to spend corresponding
transaction output is named a party **owning state**: it is a party that has the
right to change the corresponding part of the smart contract state by creating a
new *state transition* and committing to it in a transaction spending the output
containing previous state. This procedure represents **closing of a seal over
state transition**, and a pair of spending transaction and corresponding
extra-transaction data on the state transition are named **witness**.

*State transition* **assigns** *state* to a set of defined **single-use seals**.
Each smart contract may maintain different forms of state and define different
kinds of single-use seals with different validation rules. Additionally to this,
state transition may contain different metadata and *Simplicity scripts*,
defining parts of its business logic.

Which types of state, seals, metadata and which script extensions are allowed
within state transitions is defined by **schema**. Thus, schema can be seen as
validation rules for *client-side validation*; schema is always defined by
the issuer in state genesis. Schema also may contain Turing-complete *Simplicity
scripts* defining parts of the business logic for *client-side validation*.

RGB operates in "shards", where each contract has a separate **state history**
and data; different smart contracts never intersect in their histories
directly. This allows another level of scalability; and while the therm "shard"
is  incorrect, we use it to demonstrate that RGB actually achieves what was
planned to be achieved with "Ethereum shards".

While being separately maintained, RGB contracts may interact via **Spectrum**
protocol over the Lightning Network, allowing multiparty **coordinated state
changes**, which, for instance, enables functionality like DEX over Lightning
etc.

Thus, by their abilities RGB smart contracts go beyond what is possible with
Ethereum-like smart contract system, providing more layered, scalable, private
and safe approach, where the ownership of the smart contract state is separated
from the smart contract creation.

TBD:
* data confidentiality management
* state isolation

## Specification

## Compatibility

TBD:
* Bitcoin and it's future evolition
  - Schnorr signatures
  - Taproot and Tapscript
  - SIGHASH_ANY
* Lightning Network and it's future evolution
  - eltoo
  - Payment points (PTLCs)
  - Taproot/Schnorr signature adoption
  - Channel factories and multipeer channels
  - Submarine swaps
  - Channel splits
  - Bi-directional channels
  - Multipath routing
* Liquid Network
* Confidential Assets
* DLCs
* Bitcoin-compatible blockchains
* Atomic swaps

## Rationale

## Reference implementation

## Acknowledgements

RGB originally was envisioned in 2016 by Giacomo Zucco and Peter Todd,
presenting a further development for Peter’s Todd concepts of client-side
validation and single-use seals. Its development was maintained for some time by
BHB Network, inbitcoin and supported by Poseidon Group; for this time the main
developer for RGB technology was Alekos Filini. Since mid-2019 Pandora Core AG
and Dr Maxim Orlovsky have become main contributors to the technology
development; RGB became a project basing on set of standards maintained by
LNP/BP Standards Association. At this stage RGB was refactored from a token
protocol into universal smart contract system, had absorbed many parts of
confidential transactions and switched on using bulletproofs from Blockstream.
The overall work on RGB since 2019 was financially supported by Bitfinex/ Tether
Inc and Fulgur Ventures.

As a technology RGB had multiple contributors and reviewers additionally to the
mentioned persons; among them Christian Decker, Christophe Diederichs, Emil
Bayes, Fabrizio Armani, Federico Tenga, John Carvalho, Martino Salvetti, Max
Hillebrand, Marco Amadori, Martin Habovštiak, Nicola Busanello, Oleg Mikhalsky,
Olga Ukolova, Paolo Arduino, Rene Pickhardt, Reza Bandegi, Stephano Pellegrini,
ZmnSCPxj, Zoe Faltiba, and many other independent contributors

RGB acknowledges significant role which played advises of Adam Back and work of
Blockstream engineers in the design of its technology, including Andrew Poelstra
(bulletproofs, mimblewimple, confidential transactions), Peter Wuille
(confidential transactions, bulletproofs) and Christian Decker (Lightning
network, system architecture design) works.

## References

## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.
