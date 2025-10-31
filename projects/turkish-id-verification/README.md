# Turkish ID Support

**Status:** On staging  (31.10.25)
**What:** Enable Turkish national ID verification in Self for extending privacy-preserving identity proofs beyond passports
**Repo:** https://github.com/selfxyz/self/pull/1302
**Team:** @0x471
**Contact:** x.com/d0x471b

**How Self fits:** Expands Self beyond passports to Turkish national IDs. Device reads NFC chip, generates zero-knowledge proof from Turkish ID cryptographic signatures, apps verify yes/no results for policies like age and other personal information without seeing raw identity data.

## The Problem

1. Algorithm Detection Failure
The `brutforceSignatureAlgorithm` detection algorithm wasn't identifying Turkish DSCs correctly:
- Turkish Digital Signing Certificate (DSCs) uses RSA-PSS
- But the actual signature algorithm for the SOD (Statement of Designation) is legacy RSA (not RSA-PSS)
- The brutforce algorithm was failing to detect this mixed-mode configuration

2. Missing CSCA Certificate
The Turkish Country Signing Certificate Authority (CSCA) certificate was not in Self's certificate Merkle tree:
- CSCA certificates form the root of trust for verifying DSCs
- Without the Turkish CSCA in the tree, the certificate chain validation would fail
- This prevented even algorithmically-correct signatures from being verified


## The Solution

1. Enhanced `brutforceSignatureAlgorithm` in `brutForcePassportSignature.ts`:
- Added fallback logic to detect RSA-PSS DSC paired with RSA-signed SOD
- After initial RSA and RSA-PSS checks fail, tries `brutforceHashAlgorithm(passportData, 'rsa')`
- Returns signature algorithm configuration with `saltLength: 0` when RSA verification succeeds
- Maintains backward compatibility with all existing passport types

2. CSCA Certificate Addition
Added Turkish CSCA certificate to Self's Merkle tree:
- Imported Turkish CSCA root certificate into the certificate Merkle tree
- Enables proper DSC verification against Turkish national authority