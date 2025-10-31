# GCP JWT TEE Attestation Verification

**Status:** In review (PR open with requested changes) (31.10.25)
**What:** Zero-knowledge circuit for verifying Google Cloud Platform Confidential Space JWT attestations, proving code executed in a trusted execution environment with Docker by verifying the Google certificate chain and having the docker image hash and key exchange nonce as public outputs
**Repo:** https://github.com/selfxyz/self/pull/1317
**Team:** @0x471
**Contact:** x.com/d0x471b

**How Self fits:**  Improves Self's infrastructure by cryptographically verifying Google's certificate chain and extracting Docker image hash + key exchange nonce as public outputs when Self is generating proofs in TEEs.

## The Problem

Trusted Execution Environments (TEEs) like GCP Confidential Space provide hardware-enforced isolation for sensitive computations. They issue JWT attestations as cryptographic proof that code executed inside the secure enclave. The challenge is verifying these attestations were legitimately issued by GCP's TEE infrastructure in zk-circuits.

## The Solution

Implemented a zero-knowledge circuit for verifying GCP Confidential Space JWT attestations using ZKEmail's `zk-jwt` library. (https://github.com/zkemail/zk-jwt)

1. JWT Signature Verification
- Validated the JWT signature using the leaf certificate's public key

2. X.509 Certificate Chain Validation
- Validated complete 3-level certificate chain (leaf → intermediate → root)

3. TEE Claim Extraction
- Extracted Docker image hash and key exchange nonce as public outputs