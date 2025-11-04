# Post-Quantum Secure Communication

**Status:** Implementation complete, ready for security audit by [zkSecurity](https://www.zksecurity.xyz/)
**What:** Upgrade mobile ↔ TEE handshake to quantum-resistant key exchange using PQXDH
**Repos:**
- Mobile side: https://github.com/selfxyz/self/pull/1316
- TEE server side: https://github.com/selfxyz/tee-prover-server/pull/27

**Contact:** @armankolozyan (https://armankolozyan.com)

**How Self fits:** Strengthens Self's secure communication session so passport/ID verification traffic resists ["harvest now, decrypt later"](https://en.wikipedia.org/wiki/Harvest_now,_decrypt_later) quantum attacks.

## Context

Self enables users to prove their identity in a privacy-preserving manner by generating zero-knowledge proofs from their passport data. These proofs allow verification of identity attributes without revealing the underlying sensitive information.

Ideally, proof generation would happen entirely client-side on the user's mobile device. However, this approach faces practical challenges. Zero-knowledge proof systems require large cryptographic artifacts: proving keys for passport verification circuits can exceed GBs. Downloading these artifacts to mobile devices is impractical. Furthermore, proof generation itself is computationally intensive, requiring significant CPU and memory resources that would drain mobile battery life and cause poor performance on lower-end devices.

To address these constraints, Self uses a Trusted Execution Environment (TEE) to perform proof generation. The mobile app sends encrypted passport data to the TEE, which generates the zero-knowledge proof in a secure, isolated environment. The TEE provides strong isolation designed to keep passport data confidential even from the cloud provider, while its computational resources enable fast proof generation without impacting the mobile user experience.

## The Problem

This mobile ↔ TEE communication channel requires strong cryptographic protection. The original implementation used P-256 ECDH (Elliptic Curve Diffie-Hellman) to establish a shared secret during the handshake, which was then used to encrypt the passport data transmission. While P-256 ECDH is secure against current computers, it is vulnerable to quantum computers through [Shor's algorithm](https://en.wikipedia.org/wiki/Shor%27s_algorithm).

This creates a ["harvest now, decrypt later"](https://en.wikipedia.org/wiki/Harvest_now,_decrypt_later) threat: adversaries can record encrypted traffic today during the mobile ↔ TEE handshake and store it for later. When large-scale quantum computers become available, these adversaries can decrypt the captured data using Shor's algorithm to break the ECDH key exchange. The result is that user passport information and biometric data are compromised retroactively, potentially years or decades after the original communication.

## Background

Post-quantum cryptography addresses quantum threats by using mathematical problems that remain computationally hard even for quantum computers. Classical algorithms like P-256 ECDH rely on the elliptic curve discrete logarithm problem, which Shor's algorithm can solve efficiently on quantum computers. In contrast, lattice-based cryptography uses problems like Module Learning With Errors (MLWE). NIST recently standardized [ML-KEM](https://csrc.nist.gov/pubs/fips/203/final) (formerly known as Kyber), providing a thoroughly-reviewed post-quantum key encapsulation mechanism based on MLWE.

ML-KEM is a Key Encapsulation Mechanism (KEM), which works differently from traditional Diffie-Hellman key exchange. In a KEM, one party generates a keypair and publishes their public key. The other party uses this public key to perform encapsulation: they generate a random shared secret and encrypt it with the public key to produce a ciphertext. The first party then performs decapsulation: they use their private key to decrypt the ciphertext and recover the same shared secret. This asymmetric approach works well with lattice-based cryptography, where the mathematical structure naturally supports encryption operations rather than interactive key agreement.

However, post-quantum cryptography is relatively new compared to classical algorithms like ECDH, which have decades of cryptanalysis behind them. Pure post-quantum solutions carry the risk that future mathematical breakthroughs could unexpectedly weaken lattice-based assumptions. To address this, cryptographers developed hybrid constructions that combine classical and post-quantum algorithms. A hybrid protocol remains secure as long as *at least one* of its component algorithms is unbroken, providing defense-in-depth against both quantum computers and potential advances in cryptanalysis.

[PQXDH (Post-Quantum Extended Diffie-Hellman)](https://signal.org/docs/specifications/pqxdh/) is a hybrid key exchange protocol originally developed by Signal for their messaging application. The PQXDH specification combines X25519 (a widely-trusted elliptic curve Diffie-Hellman algorithm) with ML-KEM-768 (a lattice-based key encapsulation mechanism). During a PQXDH handshake, both parties perform X25519 key agreement and ML-KEM encapsulation/decapsulation, then combine the two resulting shared secrets using HKDF (a key derivation function) to produce the final session key. This ensures that breaking the session key requires breaking *both* X25519 and ML-KEM-768 simultaneously.

## The Solution

We implemented PQXDH (Post-Quantum Extended Diffie-Hellman). The implementation follows Signal's PQXDH specification, which has undergone extensive security review by the cryptographic community. To maintain backward compatibility with existing clients, we implemented suite negotiation. Clients advertise support for both Self-PQXDH-1 (the hybrid protocol) and legacy-p256 (the original P-256 ECDH). The TEE server selects the strongest mutually-supported suite, preferring PQXDH when available.

The implementation uses well-vetted cryptographic libraries on both sides. The TypeScript client relies on [@noble/post-quantum](https://github.com/paulmillr/noble-post-quantum), a zero-dependency implementation of ML-KEM. The Rust server uses the [RustCrypto ml-kem](https://github.com/RustCrypto/KEMs/tree/master/ml-kem) crate, which implements the NIST-standardized ML-KEM-768 algorithm. Both libraries follow NIST FIPS 203 (the ML-KEM standard), ensuring cross-language interoperability.

The PQXDH handshake occurs in three phases. First, the client opens a WebSocket connection and sends a hello message advertising supported cipher suites along with its X25519 public key. Second, the TEE server selects the strongest mutually-supported suite and responds with its own X25519 and Kyber public keys embedded in an attestation document. The client then performs both X25519 key agreement and Kyber encapsulation, combines the resulting shared secrets using HKDF, and sends the Kyber ciphertext back to the server in a key_exchange message. Third, the server decapsulates the Kyber ciphertext, performs the same HKDF derivation, and confirms completion. Both sides now share an identical session key derived from both classical and post-quantum shared secrets.

## Implementation Details

### Mobile Side (TypeScript)

**Pull Request:** https://github.com/selfxyz/self/pull/1316

**New Files:**

- **`common/src/utils/proving/pqxdh-crypto.ts`**
  - `generateX25519Keypair()` - generates X25519 keypairs for ECDH
  - `kyberEncapsulate()` - performs ML-KEM-768 encapsulation
  - `computeX25519SharedSecret()` - computes X25519 shared secret
  - `deriveSessionKey()` - derives session key from both X25519 and Kyber shared secrets using HKDF
  - `getSupportedSuites()` - returns supported cipher suites in preference order

- **`common/tests/pqxdh-crypto.test.ts`**
  - 15 tests covering all cryptographic primitives
  - Tests for Signal spec compliance (F prefix, zero salt, info string format)
  - End-to-end key exchange test

- **`common/tests/pqxdh-cross-language.test.ts`**
  - Cross-language integration test spawning actual Rust TEE server
  - Tests complete PQXDH handshake between TypeScript client and Rust server
  - Verifies both sides derive identical session keys
  - Tests legacy P-256 fallback, suite negotiation, and error handling
  - 5 integration tests validating protocol correctness

**Modified Files:**

- **`common/src/utils/proving/types.ts`**
  - Extended `HelloResponse` type with optional `selected_suite`, `x25519_pubkey`, `kyber_pubkey` fields
  - These fields are only present when the TEE selects `Self-PQXDH-1`

- **`packages/mobile-sdk-alpha/src/proving/provingMachine.ts`**
  - Added `selectedSuite`, `kyberCiphertext`, `sharedKey` to proving store state
  - Modified `_handleWsOpen()` to advertise supported suites in hello message
  - Modified `_handleWebSocketMessage()` to:
    - Handle suite selection from TEE
    - Perform hybrid X25519 + Kyber key exchange when PQXDH is selected
    - Send key exchange message with Kyber ciphertext
    - Wait for TEE confirmation before proceeding
    - Fall back to legacy P-256 flow if that's what the TEE selects
  - Updated `_closeConnections()` and `init()` to clean up PQXDH state

### TEE Server Side (Rust)

**Pull Request:** https://github.com/selfxyz/tee-prover-server/pull/27

**New Files:**

- **`src/lib.rs`**
  - Library entry point exposing internal modules for testing
  - Enables integration tests to access store, types, and utils

- **`examples/pqxdh_test_server.rs`**
  - Standalone test server for local PQXDH testing
  - Runs with `--features test_mode` (no DB, no circuits, mock attestation)
  - Includes `debug_get_session_key` endpoint for cross-language test verification
  - Used by TypeScript client-side integration tests to spawn real server programmatically

- **`tests/store_tests.rs`**
  - 8 comprehensive tests for LruStore key material management
  - Tests all KeyMaterial variants: LegacyP256, PqxdhPending, PqxdhComplete
  - Tests state transitions, duplicate UUID rejection, LRU eviction

**Modified Files:**

- **`Cargo.toml`**
  - Added PQXDH dependencies
  - Added `test_mode` feature flag for local testing without TEE infrastructure

- **`src/types.rs`**
  - Extended `HelloResponse` with:
    - `selected_suite: String` - negotiated cipher suite
    - `x25519_pubkey: Option<Vec<u8>>` - server's X25519 public key (PQXDH only)
    - `kyber_pubkey: Option<Vec<u8>>` - server's Kyber public key (PQXDH only)

- **`src/store.rs`**
  - Added `KeyMaterial` enum for multi-state session management
  - Added methods:
    - `get_key_material()` - retrieves KeyMaterial for state inspection
    - `update_key_material()` - transitions PqxdhPending -> PqxdhComplete
    - Updated `get_shared_secret()` - handles all KeyMaterial variants

- **`src/server.rs`**
  - Modified `hello()` RPC method:
    - Added `supported_suites` parameter
    - Implements suite negotiation (prefers Self-PQXDH-1, fallbacks to legacy-p256)
    - PQXDH branch: generates X25519 + Kyber keypairs, stores PqxdhPending state
    - Legacy branch: existing P-256 flow (unchanged)
    - Validates key sizes (X25519: 32 bytes, P-256: 33 bytes)
  - Added `key_exchange()` RPC method:
    - Validates Kyber ciphertext length (1088 bytes for ML-KEM-768)
    - Decapsulates ciphertext to recover Kyber shared secret
    - Derives session key using HKDF per Signal spec
    - Updates store: PqxdhPending -> PqxdhComplete
    - Returns "key_exchange_complete" confirmation

- **`src/utils.rs`**
  - Added `#[cfg(feature = "test_mode")]` mock attestation function:
    - Returns JWT-like structure with base64url encoding
    - NOT cryptographically valid (for testing only)
    - Allows local development without AWS Nitro Enclaves

## Testing

### TypeScript Unit Tests
```bash
yarn workspace @selfxyz/common test pqxdh-crypto
```

### Rust Unit Tests
```bash
cd tee-prover-server
cargo test
```

### Cross-Language Integration Tests
```bash
yarn workspace @selfxyz/common test pqxdh-cross-language
```

These tests spawn a real Rust TEE server programmatically and verify that the TypeScript client successfully performs PQXDH handshake with the Rust server, that both sides derive identical session keys, and that suite negotiation, legacy P-256 fallback, and error handling all function correctly.

## Future Work

The implementation is complete and all tests pass, but several audits are required before production deployment. A third-party security audit by zkSecurity is pending to review the PQXDH protocol implementation across both mobile and TEE components. Additionally, both cryptographic libraries used in this implementation, [@noble/post-quantum](https://github.com/paulmillr/noble-post-quantum) and [RustCrypto ML-KEM](https://github.com/RustCrypto/KEMs/tree/master/ml-kem), have not yet undergone formal security audits. Independent audits of these libraries are necessary to provide strong assurance of their correctness and security before deploying PQXDH to production.

## References

- [Signal PQXDH Specification](https://signal.org/docs/specifications/pqxdh/)
- [NIST FIPS 203: Module-Lattice-Based Key-Encapsulation Mechanism Standard](https://csrc.nist.gov/pubs/fips/203/final)
- [Harvest Now, Decrypt Later (Wikipedia)](https://en.wikipedia.org/wiki/Harvest_now,_decrypt_later)
- [@noble/post-quantum GitHub](https://github.com/paulmillr/noble-post-quantum)
- [RustCrypto ML-KEM Implementation](https://github.com/RustCrypto/KEMs/tree/master/ml-kem)

## Updates

- **Week 1:** Not in the residency yet. :(
- **Week 2:** Meeting with Rémi and Ayman to discuss design. Implemented mobile side.
- **Week 3:** Implemented TEE side. Written unit and integration tests.

## Contact

For questions, please contact: **@armankolozyan** (https://armankolozyan.com)
