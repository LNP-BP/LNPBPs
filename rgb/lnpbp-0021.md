```
LNPBP: 0021
Vertical: Smart contracts
Title: RGB non-fungible assets schema for collectibles (RGB-21)
Authors: Dr Maxim Orlovsky <orlovsky@protonmail.ch>,
         Carlos Roldan,
         Giacomo Zucco,
         Olgsa Ukolova
Comments-URI: <https://github.com/LNP-BP/LNPBPs/issues/70>
Status: Proposal
Type: Standards Track
Created: 2020-09-10
License: CC0-1.0
```

- [x] Unique tokens & token-specfic data
- [x] Issue control
- [x] Generic token data, internal or external, in different formats
- [x] Engravings (any why Schema subclassing is required)
- [x] LockUTXOs and descriptors for proof of reserves
- [x] Renominations
- [x] Rights splits

```rust
Schema {
    rgb_features: none!(),
    root_id: none!(),
    genesis: GenesisSchema {
        metadata: type_map! {
            FieldType::Name => Once,
            FieldType::Description => NoneOrOnce,
            FieldType::Data => NoneOrMore,
            FieldType::DataFormat => NoneOrOnce,
            // Proof of reserves UTXO
            FieldType::LockUtxo => NoneOrMore,
            // Proof of reserves scriptPubkey descriptor used for
            // verification
            FieldType::LockDescriptor => NoneOrUpTo(32),
            FieldType::Timestamp => Once,
            FieldType::NftSource => NoneOrMore,
            FieldType::Salt => Once
        },
        owned_rights: type_map! {
            OwnedRightsType::Inflation => NoneOrOnce,
            OwnedRightsType::Renomination => NoneOrOnce,
            OwnedRightsType::Ownership => OnceOrMore
        },
        public_rights: none!(),
        abi: bmap! {
            // Here we validate hash uniqueness of NftSource values and that
            // there is always one ownership state per NftSource matching
            // its hash
            // and verification of proof of reserves
            GenesisAction::Validate => Procedure::Embedded(StandardProcedure::NonfungibleInflation)
        },
    },
    extensions: bmap! {},
    transitions: type_map! {
        TransitionType::Issue => TransitionSchema {
            metadata: type_map! {
                // Proof of reserves UTXO
                FieldType::LockUtxo => NoneOrMore,
                // Proof of reserves scriptPubkey descriptor used for
                // verification
                FieldType::LockDescriptor => NoneOrUpTo(32),
                FieldType::NftSource => NoneOrMore,
                FieldType::Salt => Once
            },
            closes: type_map! {
                OwnedRightsType::Inflation => Once
            },
            owned_rights: type_map! {
                OwnedRightsType::Inflation => NoneOrOnce,
                OwnedRightsType::Ownership => OnceOrMore
            },
            public_rights: none!(),
            abi: bmap! {
                // Here we validate hash uniqueness of NftSource values and that
                // there is always one ownership state per NftSource matching
                // its hash, plus the fact that
                // count(in(inflation)) >= count(out(inflation), out(nft_source))
                // and verification of proof of reserves
                TransitionAction::Validate => Procedure::Embedded(StandardProcedure::NonfungibleInflation)
            }
        },
        TransitionType::Transfer => TransitionSchema {
            metadata: type_map! {
                // By default, use 0
                FieldType::Salt => Once
            },
            closes: type_map! {
                OwnedRightsType::Ownership => OnceOrMore
            },
            owned_rights: type_map! {
                OwnedRightsType::Ownership => OnceOrMore
            },
            public_rights: none!(),
            abi: none!()
        },
        // One engraving per set of tokens
        TransitionType::Engraving => TransitionSchema {
            metadata: type_map! {
                FieldType::Data => NoneOrMore,
                FieldType::DataFormat => NoneOrOnce,
                // By default, use 0
                FieldType::Salt => Once
            },
            closes: type_map! {
                OwnedRightsType::Ownership => OnceOrMore
            },
            owned_rights: type_map! {
                OwnedRightsType::Ownership => OnceOrMore
            },
            public_rights: none!(),
            abi: none!()
        },
        TransitionType::Renomination => TransitionSchema {
            metadata: type_map! {
                FieldType::Name => NoneOrOnce,
                FieldType::Description => NoneOrOnce,
                FieldType::Data => NoneOrMore,
                FieldType::DataFormat => NoneOrOnce
            },
            closes: type_map! {
                OwnedRightsType::Renomination => Once
            },
            owned_rights: type_map! {
                OwnedRightsType::Renomination => NoneOrOnce
            },
            public_rights: none!(),
            abi: none!()
        },
        // Allows split of rights if they were occasionally allocated to the
        // same UTXO, for instance both assets and issuance right. Without
        // this type of transition either assets or inflation rights will be
        // lost.
        TransitionType::RightsSplit => TransitionSchema {
            metadata: type_map! {
                FieldType::Salt => Once
            },
            closes: type_map! {
                OwnedRightsType::Inflation => NoneOrMore,
                OwnedRightsType::Ownership => NoneOrMore,
                OwnedRightsType::Renomination => NoneOrOnce
            },
            owned_rights: type_map! {
                OwnedRightsType::Inflation => NoneOrMore,
                OwnedRightsType::Ownership => NoneOrMore,
                OwnedRightsType::Renomination => NoneOrOnce
            },
            public_rights: none!(),
            abi: bmap! {
                // We must allocate exactly one or none rights per each
                // right used as input (i.e. closed seal); plus we need to
                // control that sum of inputs is equal to the sum of outputs
                // for each of state types having assigned confidential
                // amounts
                TransitionAction::Validate => Procedure::Embedded(StandardProcedure::RightsSplit)
            }
        }
    },
    field_types: type_map! {
        FieldType::Name => DataFormat::String(256),
        FieldType::Description => DataFormat::String(core::u16::MAX),
        FieldType::Data => DataFormat::Bytes(core::u16::MAX),
        FieldType::DataFormat => DataFormat::Unsigned(Bits::Bit16, 0, core::u16::MAX as u128),
        // While UNIX timestamps allow negative numbers; in context of RGB
        // Schema, assets can't be issued in the past before RGB or Bitcoin
        // even existed; so we prohibit all the dates before RGB release
        // This timestamp is equal to 10/10/2020 @ 2:37pm (UTC) - the same
        // as for RGB-20 standard.
        FieldType::Timestamp => DataFormat::Integer(Bits::Bit64, 1602340666, core::i64::MAX as i128),
        FieldType::LockUtxo => DataFormat::TxOutPoint,
        FieldType::LockDescriptor => DataFormat::String(core::u16::MAX),
        FieldType::BurnUtxo => DataFormat::TxOutPoint,
        // This type is used to "shift" unique tokens ids if there was a
        // collision between them
        FieldType::Salt => DataFormat::Unsigned(Bits::Bit32, 0, core::u32::MAX as u128),
        // Hash of these data serves as a unique NFT identifier;
        // if NFT contains no intrinsic data than simply put any unique
        // value here (like counter value, increased with each token);
        // it must be unique only within single issuance transition
        FieldType::NftSource => DataFormat::Bytes(core::u16::MAX)
    },
    owned_right_types: type_map! {
        OwnedRightsType::Inflation => StateSchema {
            // How much issuer can issue tokens on this path
            format: StateFormat::CustomData(DataFormat::Unsigned(Bits::Bit64, 0, core::u64::MAX as u128)),
            abi: none!()
        },
        OwnedRightsType::Ownership => StateSchema {
            // This is unique token identifier, which is
            // SHA256(SHA256(nft_source_state), issue_transition_id)
            // convoluted to 32-bits with XOR operation and then XORed with
            // salt value from state transition metadata.
            // NB: It is unique inside single state transition only, not
            // globally. For global unique id use non-convoluted hash value.
            format: StateFormat::CustomData(DataFormat::Unsigned(Bits::Bit32, 0, core::u32::MAX as u128)),
            abi: bmap! {
                // Here we ensure that each unique state value is
                // transferred once and only once (using "salt" value for
                // collision resoultion)
                AssignmentAction::Validate => Procedure::Embedded(StandardProcedure::IdentityTransfer)
            }
        },
        OwnedRightsType::Renomination => StateSchema {
            format: StateFormat::Declarative,
            abi: none!()
        }
    },
    public_right_types: none!(),
}
```
