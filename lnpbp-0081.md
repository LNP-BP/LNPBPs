```
LNPBP: 0081
Layer: Client-validated data (3)
Vertical: Cryptographic primitives
Title: Tagged merkle trees for client-side-validation
Author: Dr Maxim Orlovsky  <orlovsky@protonmail.ch>,
        Peter Todd
Comments-URI: https://github.com/LNP-BP/lnpbps/issues/<____>
Status: Draft
Type: Standards Track
Created: 2021-05-11
License: CC0-1.0
```

## Abstract

## Background

## Motivation

Problems with modern merkle trees:
- Depth extension attack: no commitment to the tree depth

## Design

Based on bitcoin merklization with following modifications:
- Tagged hashing for source data
- Tagged hashing of each tree object
- Commitments to depth, width and height of the tree
- Custom placeholders for empty objects
- Restricting tree source to 2^16 elements (height is always <=16)

## Specification

## Compatibility

## Rationale

## Reference implementation



## Acknowledgements

## References

## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal license.

## Test vectors
