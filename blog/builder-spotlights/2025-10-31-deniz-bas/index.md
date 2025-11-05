---
title: "Builder Spotlight: Deniz Bas"
date: "2025-10-31"
author: "d0x471b"
project: "Turkish ID Support & Self Infrastructure"
links:
  repo: ""
  demo: ""
  video: ""
  contact: "https://x.com/d0x471b"
self:
  proofs: ["citizenship", "age", "residency"]
  flow: ""
  verifier_learns: ""
  sdks: [""]
stack: ["Swift", "TypeScript", "Circom", "Solidity", "Core NFC", "Hyperlane", "GCP", "TEEs", "JWTs"]
---

## Building infrastructure that matters.

### 1. Turkish Local ID Support in Self

Problem: Turkish national IDs use RSA-PSS DSCs (Digital Signing Certificates) but sign SODs (Statement of Designation) with legacy RSA, a mixed-mode configuration that Self's algorithm detection didn't recognize. Additionally, Turkey's CSCA root certificate wasn't in Self's Merkle tree.

Solution: Enhanced `brutforceSignatureAlgorithm` to detect RSA-PSS/RSA combinations and added Turkey's CSCA certificate to the trust tree. [PR #1302](https://github.com/selfxyz/self/pull/1302) merged and deployed to TestFlight.

Impact: 80M+ Turkish citizens can now use national IDs for privacy-preserving verification without exposing name, ID number, or any other personal informaiton.

### 2. GCP JWT TEE Attestation Verification

Problem: Google Cloud Platform Confidential Space issues JWT attestations proving code executed in hardware-isolated TEEs (Trusted Execution Environments). It's used by Self for proof generation. This contribution makes it provable by proving the TEE JWT attestation in a zk-circuit and public output the docker image hash and key exchange nonce.

Solution: Implemented a zero-knowledge circuit using ZKEmail's `zk-jwt` library that verifies JWT signatures, validates 3-level X.509 certificate chains (leaf -> intermediate -> root CA), and extracts Docker image hash + key exchange nonce as minimal public outputs. [PR #1317](https://github.com/selfxyz/self/pull/1317) currently in review.

Impact: Enables verification that specific code (via Docker image hash) ran in Google's secure hardware without revealing the full JWT attestation. Makes Self's infrastructure verifiable when generating proofs in TEEs, allowing applications to confirm "this proof came from verified secure enclave code" with cryptographic certainty.

### 3. DSC Certificate Parser (Developer Tooling App)

Problem: CSCA (Country Signing Certificate Authority) certificates are difficult to find online beyond the ICAO master list. When implementing NFC-based identity verification, developers need to know which CSCA certificate corresponds to a specific passport or national ID, but there's no easy way to identify this from the document itself.

Solution: Built a Swift mobile app that scans NFC chips from identity documents, extracts the DSC certificate, and displays the issuer name and details. This reveals which CSCA certificate is needed, allowing developers to search for and obtain the correct certificate by name for their verification implementation. [GitHub: 0x471/dsc-parser](https://github.com/0x471/dsc-parser)

Impact: Reduces friction in CSCA certificate discovery for identity verification developers. Scan a document, see DSC issuer details, search by name online. Doesn't fully solve the discovery problem (certificates still need to be found manually), but makes identifying which CSCA to look for significantly easier.


### 4. Celo & Base Messaging (Hyperlane)

Problem: Self proofs are verified on Celo, but applications on other chains (like Base) need access to verification results. No reference implementation existed for bridging identity proof verification across chains or demonstrating how cross-chain messaging works for multi-chain identity scenarios.

Solution: Built a reference implementation using Hyperlane's interoperability protocol to send messages between Celo and Base. Smart contracts demonstrate deployment and message passing for cross-chain scenarios. [GitHub: 0x471/celo-base-interop](https://github.com/0x471/celo-base-interop)

Impact: Reference architecture for multi-chain identity applications. Enables use cases where identity proofs are verified on different chains than generation (e.g., prove citizenship on Celo, access DeFi on Base).
