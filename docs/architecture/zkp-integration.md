# Zero-Knowledge Proof System

`savitri-zkp` provides a multi-backend zero-knowledge proof framework for block integrity verification and monolith commitments.

## Backends

| Backend | Feature Flag | Library | Use Case |
|---------|-------------|---------|----------|
| **Mock** | `mock` | None | Development and testing |
| **PLONK** | `plonk` | halo2_proofs | General-purpose ZKP |
| **Arkworks** | `arkworks` | ark-bn254, Groth16 | Production (BN254 curve) |

Default: no backend enabled. The `mock` backend is used for development.

**Safety**: The system panics if a production backend is requested without the corresponding feature flag. It never silently falls back to MockVerifier.

## Configuration

```rust
pub struct ZkpConfig {
    pub backend: ZkpBackend,
    pub max_proof_size: usize,          // bytes
    pub verification_timeout_ms: u64,
}

// Presets
ZkpConfig::development()  // Mock, 1MB, 5s
ZkpConfig::testing()      // Mock, 512KB, 1s
ZkpConfig::production()   // Arkworks, 4MB, 15s
```

## Core Types

### Statement

A statement contains the public inputs for a proof:

```rust
pub struct Statement {
    pub a: [u8; 32],   // commitment field
    pub b: [u8; 32],   // commitment field
    pub c: [u8; 32],   // commitment field
    pub d: [u8; 32],   // commitment field
    pub e: u64,         // scalar field
    pub f: u64,         // scalar field
}
```

### ZkProof

```rust
pub struct ZkProof {
    pub proof: Vec<u8>,
    pub public_inputs: Vec<u8>,
    pub verification_key: Vec<u8>,
}
```

## Verifier API

```rust
pub trait ZkVerifier: Send + Sync {
    fn verify(&self, statement: &Statement, proof: &ZkProof) -> Result<bool>;
    fn batch_verify(&self, statements: &[Statement], proofs: &[ZkProof]) -> Result<Vec<bool>>;
}
```

### Creating a Verifier

```rust
use savitri_zkp::{create_verifier, ZkpConfig, ZkpBackend};

// Development
let verifier = create_verifier(ZkpConfig::development())?;

// Production (requires `arkworks` feature)
let verifier = create_verifier(ZkpConfig::production())?;

// Verify a proof
let is_valid = verifier.verify(&statement, &proof)?;

// Batch verify
let results = verifier.batch_verify(&statements, &proofs)?;
```

## Prover API

```rust
pub trait ZkProver: Send + Sync {
    fn prove(&self, statement: &Statement) -> Result<ZkProof>;
}
```

### Arkworks Prover

```rust
// Trusted setup (generates proving + verification keys)
let prover = ArkworksProver::from_setup()?;

// From existing keys
let prover = ArkworksProver::from_keys(pk_bytes, vk_bytes)?;

// Generate proof
let proof = prover.prove(&statement)?;
```

The Arkworks prover uses a deterministic seed (`SHA-256("savitri-monolith-sum-circuit-setup-v1")`) for reproducible trusted setup.

## Monolith Integration

ZKP is used to verify monolith block commitments:

### Monolith Header

```rust
pub struct MonolithHeader {
    pub headers_commit: [u8; 64],   // SHA-512 binary tree of block headers
    pub state_commit: [u8; 64],     // state snapshot commitment
    pub exec_height: u64,           // execution height
    pub epoch_id: u64,              // epoch identifier
}
```

### Headers Commitment

Block headers are committed using a binary tree reduction:

```
leaf_0 = H(block_header_0)
leaf_1 = H(block_header_1)
...
node_01 = SHA-512(leaf_0 || leaf_1)
node_23 = SHA-512(leaf_2 || leaf_3)
...
root = SHA-512(node_01..n-1 || node_23..n)
```

### Monolith Commitment

The full commitment binds all monolith fields:

```
commitment = SHA-256(
    headers_commit || state_commit || exec_height || epoch_id || prev_state_root
)
```

### Verification

```rust
use savitri_zkp::monolith::monolith_zkp;

// Verify a monolith proof
let is_valid = monolith_zkp::verify_monolith_proof(
    &verifier,
    &monolith_header,
    &proof,
    prev_epoch_id,
)?;

// Generate a monolith proof
let proof = monolith_zkp::generate_monolith_proof(
    &prover,
    &monolith_header,
)?;
```

## Feature Flags

```toml
[features]
default = []
mock = []                    # MockVerifier (always-valid)
plonk = ["halo2_proofs"]     # PLONK backend
arkworks = ["ark-bn254", "ark-groth16", "ark-snark", "ark-relations", "ark-serialize", "ark-std"]
circom = []                  # Reserved for future Circom support
production = ["arkworks"]    # Production alias
advanced = ["plonk", "arkworks"]
all_backends = ["mock", "plonk", "arkworks"]
```

## Dependencies

| Dependency | Backend | Version |
|------------|---------|---------|
| `ark-bn254` | Arkworks | Latest |
| `ark-groth16` | Arkworks | Latest |
| `halo2_proofs` | PLONK | Latest |
| `sha2`, `sha3`, `blake3` | All | Core hashing |
| `bincode` | All | Serialization |

## No savitri-core Dependency

`savitri-zkp` has **zero internal Savitri dependencies** to avoid circular dependency chains. It communicates with other crates via the `Statement` and `ZkProof` types.
