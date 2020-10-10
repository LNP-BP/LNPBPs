```
LNPBP: 0020
Layer: Application (5)
Vertical: Smart contracts
Title: RGB fungible assets schema (RGB-20)
Authors: Dr Maxim Orlovsky <orlovsky@protonmail.ch>,
         Giacomo Zucco,
         Marco Amadori,
         Nicola Busanello,
         Federico Tenga,
         Sabina Sachtachtinskagia,
         Martino Salvetti
Comments-URI: <https://github.com/LNP-BP/LNPBPs/issues/70>
Status: Final
Type: Standards Track
Created: 2019-09-23
Finalized: 2020-10-10
License: CC0-1.0
```

Schema ID: `sch1lp4jj5xlnvkuzxgl9lrjsahxczex5x90s9j3s2ak3r4zk72akf5s8dqzun`

Encoded schema data: `schema1qxj49kgdcgcqcl2d2g5rwc5zemqn9ryzmzjgt0lk5lr3ptxrqszfhdxy85jzuju4n607mearc9deh5m8ygzrqzq53j634e8vez9ld47ns9f6evmrqfzx6kwrm93gg2c32uy2mtdpkuj2ddqfylfksdjl93t7kn2gafjqn49szpyzjw4q48qqvxfnhy95cq98sgnhss096lffudg7y63jeumham8fv75cpkhn2m2656cx643qfwwx6j2anp5dfxgsw2zmzl0ymzyucyywlej8304qugcvxnq7fjjwtldkvad28uxf5h0mnunadgu40hn40gfaqcacls36nexecl2x7pxr4ll2k58eh8gttrce8sqqah9tnl`

Schema source:
```rust
Schema {
    rgb_features: features::FlagVec::new(),
    root_id: Default::default(),
    field_types: type_map! {
        // Rational: if we will use just 26 letters of English alphabet (and
        // we are not limited by them), we will have 26^8 possible tickers,
        // i.e. > 208 trillions, which is sufficient amount
        FieldType::Ticker => DataFormat::String(8),
        FieldType::Name => DataFormat::String(256),
        // Contract text may contain URL, text or text representation of
        // Ricardian contract, up to 64kb. If the contract doesn't fit, a
        // double SHA256 hash and URL should be used instead, pointing to
        // the full contract text, where hash must be represented by a
        // hexadecimal string, optionally followed by `\n` and text URL
        FieldType::ContractText => DataFormat::String(core::u16::MAX),
        FieldType::Precision => DataFormat::Unsigned(Bits::Bit8, 0, 18u128),
        // We need this b/c allocated amounts are hidden behind Pedersen
        // commitments
        FieldType::IssuedSupply => DataFormat::Unsigned(Bits::Bit64, 0, core::u64::MAX as u128),
        // Supply in either burn or burn-and-replace procedure
        FieldType::BurnedSupply => DataFormat::Unsigned(Bits::Bit64, 0, core::u64::MAX as u128),
        // While UNIX timestamps allow negative numbers; in context of RGB
        // Schema, assets can't be issued in the past before RGB or Bitcoin
        // even existed; so we prohibit all the dates before RGB release
        // This timestamp is equal to 10/10/2020 @ 2:37pm (UTC)
        FieldType::Timestamp => DataFormat::Integer(Bits::Bit64, 1602340666, core::i64::MAX as i128),
        FieldType::HistoryProof => DataFormat::Bytes(core::u16::MAX),
        FieldType::HistoryProofFormat => DataFormat::Enum(HistoryProofFormat::all()),
        FieldType::BurnUtxo => DataFormat::TxOutPoint
    },
    owned_right_types: type_map! {
        OwnedRightsType::Inflation => StateSchema {
            // How much issuer can issue tokens on this path. If there is no
            // limit, than `core::u64::MAX` / sum(inflation_assignments)
            // must be used, as this will be a de-facto limit to the
            // issuance
            format: StateFormat::CustomData(DataFormat::Unsigned(Bits::Bit64, 0, core::u64::MAX as u128)),
            // Validation involves other state data, so it is performed
            // at the level of `issue` state transition
            abi: bmap! {}
        },
        OwnedRightsType::Assets => StateSchema {
            format: StateFormat::DiscreteFiniteField(DiscreteFiniteFieldFormat::Unsigned64bit),
            abi: bmap! {
                // sum(inputs) == sum(outputs)
                AssignmentAction::Validate => Procedure::Embedded(StandardProcedure::NoInflationBySum)
            }
        },
        OwnedRightsType::Epoch => StateSchema {
            format: StateFormat::Declarative,
            abi: bmap! {}
        },
        OwnedRightsType::BurnReplace => StateSchema {
            format: StateFormat::Declarative,
            abi: bmap! {}
        },
        OwnedRightsType::Renomination => StateSchema {
            format: StateFormat::Declarative,
            abi: bmap! {}
        }
    },
    public_right_types: bset! [],
    genesis: GenesisSchema {
        metadata: type_map! {
            FieldType::Ticker => Once,
            FieldType::Name => Once,
            FieldType::ContractText => NoneOrOnce,
            FieldType::Precision => Once,
            FieldType::Timestamp => Once,
            FieldType::IssuedSupply => Once
        },
        owned_rights: type_map! {
            OwnedRightsType::Inflation => NoneOrUpTo(None),
            OwnedRightsType::Epoch => NoneOrOnce,
            OwnedRightsType::Assets => NoneOrUpTo(None),
            OwnedRightsType::Renomination => NoneOrOnce
        },
        public_rights: bset![],
        abi: bmap! {},
    },
    extensions: bmap![],
    transitions: type_map! {
        TransitionType::Issue => TransitionSchema {
            metadata: type_map! {
                FieldType::IssuedSupply => Once
            },
            closes: type_map! {
                OwnedRightsType::Inflation => Once
            },
            owned_rights: type_map! {
                OwnedRightsType::Inflation => NoneOrUpTo(None),
                OwnedRightsType::Epoch => NoneOrOnce,
                OwnedRightsType::Assets => NoneOrUpTo(None)
            },
            public_rights: bset! [],
            abi: bmap! {
                // sum(in(inflation)) >= sum(out(inflation), out(assets))
                TransitionAction::Validate => Procedure::Embedded(StandardProcedure::FungibleInflation)
            }
        },
        TransitionType::Transfer => TransitionSchema {
            metadata: type_map! {},
            closes: type_map! {
                OwnedRightsType::Assets => OnceOrUpTo(None)
            },
            owned_rights: type_map! {
                OwnedRightsType::Assets => NoneOrUpTo(None)
            },
            public_rights: bset! [],
            abi: bmap! {}
        },
        TransitionType::Epoch => TransitionSchema {
            metadata: type_map! {},
            closes: type_map! {
                OwnedRightsType::Epoch => Once
            },
            owned_rights: type_map! {
                OwnedRightsType::Epoch => NoneOrOnce,
                OwnedRightsType::BurnReplace => NoneOrOnce
            },
            public_rights: bset! [],
            abi: bmap! {}
        },
        TransitionType::Burn => TransitionSchema {
            metadata: type_map! {
                FieldType::BurnedSupply => Once,
                // Normally issuer should aggregate burned assets into a
                // single UTXO; however if burn happens as a result of
                // mistake this will be impossible, so we allow to have
                // multiple burned UTXOs as a part of a single operation
                FieldType::BurnUtxo => OnceOrUpTo(None),
                FieldType::HistoryProofFormat => Once,
                FieldType::HistoryProof => NoneOrUpTo(None)
            },
            closes: type_map! {
                OwnedRightsType::BurnReplace => Once
            },
            owned_rights: type_map! {
                OwnedRightsType::BurnReplace => NoneOrOnce
            },
            public_rights: bset! [],
            abi: bmap! {
                TransitionAction::Validate => Procedure::Embedded(StandardProcedure::ProofOfBurn)
            }
        },
        TransitionType::BurnAndReplace => TransitionSchema {
            metadata: type_map! {
                FieldType::BurnedSupply => Once,
                // Normally issuer should aggregate burned assets into a
                // single UTXO; however if burn happens as a result of
                // mistake this will be impossible, so we allow to have
                // multiple burned UTXOs as a part of a single operation
                FieldType::BurnUtxo => OnceOrUpTo(None),
                FieldType::HistoryProofFormat => Once,
                FieldType::HistoryProof => NoneOrUpTo(None)
            },
            closes: type_map! {
                OwnedRightsType::BurnReplace => Once
            },
            owned_rights: type_map! {
                OwnedRightsType::BurnReplace => NoneOrOnce,
                OwnedRightsType::Assets => OnceOrUpTo(None)
            },
            public_rights: bset! [],
            abi: bmap! {
                TransitionAction::Validate => Procedure::Embedded(StandardProcedure::ProofOfBurn)
            }
        },
        TransitionType::Renomination => TransitionSchema {
            metadata: type_map! {
                FieldType::Ticker => NoneOrOnce,
                FieldType::Name => NoneOrOnce,
                FieldType::ContractText => NoneOrOnce,
                FieldType::Precision => NoneOrOnce
            },
            closes: type_map! {
                OwnedRightsType::Renomination => Once
            },
            owned_rights: type_map! {
                OwnedRightsType::Renomination => NoneOrOnce
            },
            public_rights: bset! [],
            abi: bmap! {}
        },
        // Allows split of rights if they were occasionally allocated to the
        // same UTXO, for instance both assets and issuance right. Without
        // this type of transition either assets or inflation rights will be
        // lost.
        TransitionType::RightsSplit => TransitionSchema {
            metadata: type_map! {},
            closes: type_map! {
                OwnedRightsType::Inflation => NoneOrUpTo(None),
                OwnedRightsType::Assets => NoneOrUpTo(None),
                OwnedRightsType::Epoch => NoneOrOnce,
                OwnedRightsType::BurnReplace => NoneOrOnce,
                OwnedRightsType::Renomination => NoneOrOnce
            },
            owned_rights: type_map! {
                OwnedRightsType::Inflation => NoneOrUpTo(None),
                OwnedRightsType::Assets => NoneOrUpTo(None),
                OwnedRightsType::Epoch => NoneOrOnce,
                OwnedRightsType::BurnReplace => NoneOrOnce,
                OwnedRightsType::Renomination => NoneOrOnce
            },
            public_rights: bset! [],
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
}
```

## Subschemata

- https://github.com/LNP-BP/LNPBPs/issues/44

## Rationale

Include from
- https://github.com/LNP-BP/LNPBPs/issues/27
- https://github.com/LNP-BP/LNPBPs/issues/28
- https://github.com/LNP-BP/LNPBPs/issues/50
