```
LNPBP: 0035
Layer: OSI Application (i7)
Vertical: Lightning network protocol
Title: Bitforst: LN message extensions for RGB data propagation
Authors: Dr Maxim Orlovsky <orlovsky@lnp-bp.org>
Comments-URI: https://github.com/LNP-BP/lnpbps
Status: Draft
Type: Standards Track
Created: 2021-03-13
Finalized: not yet
License: CC0-1.0
```

## Specifications

Tasks to cover:
1. Providing receiver with consignment. Must include:
   - Normal payments
   - LN channel funding
2. Providing & getting encrypted consignment to/from a public Bifrost node
3. Querying known assets
4. Providing asset data
5. Quering & providing public asset history

### LN messages

#### `get_consignment`

Parameters:
1. Blinded UTXO

#### `consignment`

Either spontaneous (during direct transfers) or in response to `get_consignment`
1. Encryption flag
2. Blinded UTXO (optional)
3. Consignment id
4. Fragment no
5. Total fragments
6. Partial consignment
7. Optional channel id

All data may be encrypted with the receiver blinded UTXO or descriptor first
public key

#### `known_contracts`

Parameters: list of known contract ids

#### `get_contracts`

Parameters: list of contract ids

#### `contract_info`

In response to: `get_contracts`
Parameters:
1. Genesis
2. Issuer information & signature
   - Name of the issuer
   - Domain (Onion or DNS) of the issuer
   - Public key of the issuer
   - Signature over (name, domain, genesis_id)

   or

   - RGB-23 issuer genesis?
   - Audit log (RGB-23) outpoint (single-use-seal)
3. Issuer notarization (list)
   - Name of notary
   - Domain (Onion or DNS) of notary
   - Public key of notary
   - Signature over (name, domain, issuer_signature)

   or

   - RGB-23 notary genesis
   - Audit log (RGB-23) outpoint (single-use-seal)

If all data does not fit a single message, multiple messages may be used.
RGB-23 data may be queried with normal `disclosure_contracts`/
`contract_disclosure` flow.

#### `disclose_contracts`

Parameters: list of outpoints at which known contract history ends

#### `contract_history_item`

In response to `get_contract_history`
Parameters:
1. Contract id
2. State transition
3. Anchor (fully concealed apart from the state transition)

2 & 3 may be in a form of unsigned disclosure, and in this case cover multiple
assets. Use name `contract_disclosure` in this case.
If the resulting disclosure does not fit into a single LN message, prepare
/provide multiple disclosures


Pushes consignment to receiver, unencrypted
