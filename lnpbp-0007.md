```yaml
LNPBP: 0007
Vertical: Cryptographic primitives
Title: Commitments for structural and hierarchical data
Author: Dr Maxim Orlovsky <orlovsky@lnp-bp.org>
Comments-URI: https://github.com/LNP-BP/LNPBPs/discussions/<____>
Status: Draft
Type: Standards Track
Created: 2023-01-31
Updated: 2023-01-31
Finalized: ~
Copyright: (0) public domain
License: CC0-1.0
```

- [Abstract](#abstract)
- [Background](#background)
- [Motivation](#motivation)
- [Design](#design)
- [Specification](#specification)
  - [General requirements](#general-requirements)
  - [Structured data](#structured-data)
  - [Collections](#collections)
  - [Conceal-reveal subprotocols](#conceal-reveal-subprotocols)
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

Commitments to structured and hierarchical data can be in multiple ways. 
Client-side-validation applications operate with hierarchical and structured 
data a lot, but today, no common standard exists.

## Design

Design goals:
1. Determinism: any two values which differ in their strict-serialized byte
   representation must produce different commitments.
2. Commit to data semantics (i.e. names or tags for data fields). For instance, 
   a byte can be an ordinal number, ASCII character, number of elements or set 
   of flags - and the commitment for all three items must differ.
3. Merkelize commitments for collection types, such that only a part of a 
   collection may be revealed for a verification - and the rest will be kept
   hidden.
4. Portability, or platform-independence: it must be compatible with different
   computing architectures, instruction set architectures and in networking
   systems without modification.
5. Resource limits: any data can be accessed and computed on a limited-resource
   systems (embedded systems).
6. Make protocol working with multiple hash functions.
7. Support conceal and reveal operations for specific parts of a data hierarchy.

Commitment scheme must support all [generalized algebraic data types (GADT)
used in strict types][strict type gadt]. It also must support commitments to 
both data type memory layout and semantics, as it is supported in 
[strict types].


## Specification

### General requirements

Commitment must include:
- **semantic id** of the data type ([`SemId`]) provided by strict types library, 
  which commits to type name, its memory layout and semantics (field and variant
  names and tags);
- **value commitment** which includes serialized form of all data which are a
  part of a specific instance of the data type; data serialization must be
  performed according to the [strict encoding] rules;
- **size of the data**: all atomic data types which have a variable size must
  commit to the size of the value or a number of collection items.

All signed and unsigned integers, including hash types and fixed-type data,
must be encoded in little endian order. Float numbers must be encoded according
to the appropriate IEEE or other float number standards.


### Structured data

All structured data has the following order of serialization into the hasher:
- type commitment
- value commitment;

Type commitment consists of:
- type library id;
- type `SemId` (semantic id, see above);

For sum types (enums and unions), value commitment is:
- variant tag byte (defined according to [strict types];
- single value (see below).

For tuple types, value commitment is:
- number of fields (byte);
- series of values for each field, according to their order within the type.

For structs types, value commitment is:
- number of fields (byte);
- series of field name-value pairs for each field, 
  according to their order within the type.

Field names are always shorter than 256 bytes and are serialized as a 
single-byte name length, followed by the name itself.

Values are serialized according to the strict encoding rules for that value.
Fixed-length values thus are not prefixed with the length; while variable-length
values (such as collection types) may have length prefix according to the 
specific collection type serialization rules.

Enums:
```
+=====================++==================+
| Type commitment     || Value commitment |
+----------+----------++------------------+
| 32 bytes | 32 bytes || 1 byte           |
+----------+----------++------------------+
| LibId    | SemId    || Variant tag      |
+==========+==========++==================+
```

Unions:
```
+=====================++=======================================+
| Type commitment     || Value commitment                      |
+----------+----------++----------------+----------------------+
| 32 bytes | 32 bytes || 1 byte         | Type-specific length |
+----------+----------++----------------+----------------------+
| LibId    | SemId    || Variant tag    | Value                |
+==========+==========++================+======================+
```

Tuples:
```
+=====================++=============================================+
| Type commitment     || Value commitment                            |
+----------+----------++--------------++----------------------++-----+
| 32 bytes | 32 bytes || 1 byte       || Type-specific length || ... |
+----------+----------++--------------++----------------------++-----+
| LibId    | SemId    || No of fields || Value                || ... |
+==========+==========++==============++======================++=====+
                                      <-  repeated for each field  -> 
```

Structs:
```
+=====================++=========================================================================+
| Type commitment     || Value commitment                                                        |
+----------+----------++--------------++-------------+-------------+----------------------++-----+
| 32 bytes | 32 bytes || 1 byte       || 1 byte      | Name length | Type-specific length || ... |
+----------+----------++--------------++-------------+-------------+----------------------++-----+
| LibId    | SemId    || No of fields || Name length | Field name  | Value                || ... |
+==========+==========++==============++=============+=============+======================++=====+
                                       <-            repeated for each field             -> 
```


### Collections

#### String types

#### Fixed-size arrays

#### Lists and sets

#### Maps


### Conceal-reveal subprotocols


## Compatibility


## Rationale


## Reference implementation


## Acknowledgements


## References


## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.


## Test vectors


[strict types]: https://strict-types.org/
[strict type gadt]: https://www.strict-types.org/type-system/data-primitives
[`SemId`]:
[strict encoding]: 
