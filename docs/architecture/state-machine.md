# Core Types and Cryptography

`savitri-core` provides the foundational types, cryptographic primitives, and utilities used across all Savitri Network crates.

## Core Types

### Transaction

```rust
pub struct Transaction {
    pub from: String,    // sender address (64 hex chars)
    pub to: String,      // recipient address (64 hex chars)
    pub amount: u64,     // transfer amount
}
```

### Account

```rust
pub struct Account {
    pub balance: u128,   // balance in smallest unit (18 decimals)
    pub nonce: u64,      // transaction counter
}
```

**Encoding**: Fixed 24-byte format (16 bytes balance LE + 8 bytes nonce LE). Backward compatible with old 16-byte format (balance only).

**Safety**: `credit()` and `debit()` use checked arithmetic to prevent overflow/underflow.

### Fee Limits

```rust
pub struct FeeLimits {
    pub min_fee: u128,   // default: 0.0001 SAVT (10^14)
    pub max_fee: u128,   // default: 1.0 SAVT (10^18)
}
```

## Cryptographic Primitives

### Signatures

| Algorithm | Library | Usage |
|-----------|---------|-------|
| Ed25519 | `ed25519-dalek` | Transaction signing, block signing, peer identity |

Key sizes:
- Private key: 32 bytes
- Public key: 32 bytes
- Signature: 64 bytes
- Address: hex-encoded public key (64 hex characters)

### Hash Functions

| Function | Library | Usage |
|----------|---------|-------|
| SHA-256 | `sha2` | Transaction signing message hash |
| SHA-512 | `sha2` | Block hash (with domain tag `"BLK"` + version byte) |
| BLAKE3 | `blake3` | State root computation, fast hashing |
| Keccak256 | `sha3` | Contract storage slot derivation |

### Domain-Tagged Hashing

To prevent cross-domain hash collisions, all hashes include domain tags:

| Domain | Tag | Description |
|--------|-----|-------------|
| Block | `"BLK"` + version byte | Block hash |
| Transaction root | `"TXv1"` leaf tag | Rolling accumulator over bincode |
| State root | `"STATEv1-LE"` seed | Lexicographic DB snapshot |

### Key Management

```rust
pub struct KeyManager {
    // Load or generate identity keypair
    pub fn load_or_generate(path: &Path) -> Result<KeyPair>;
    // Generate fresh keypair
    pub fn generate() -> KeyPair;
}

pub struct KeyPair {
    pub signing_key: SigningKey,
    pub verifying_key: VerifyingKey,
}
```

Keys are persisted to disk in raw 32-byte format. The `MemoryKeyStorage` provides in-memory storage for testing.

### Encryption

AES-GCM encryption for sensitive data at rest:

```rust
// Encrypt with password-derived key
let encrypted = encrypt_aes_gcm(plaintext, password)?;

// Decrypt
let plaintext = decrypt_aes_gcm(ciphertext, password)?;
```

## Slot Scheduler

Deterministic slot scheduling for leader rotation:

```rust
pub struct SlotScheduler {
    slot_duration_ms: u64,
    base_ms: u64,
    validators: Vec<String>,
    local_id: String,
}

pub enum SlotRole {
    Leader,    // produces blocks in this slot
    Follower,  // validates blocks
    Observer,  // watches only (guardian nodes)
}

pub struct SlotInfo {
    pub slot: u64,
    pub round: u32,
    pub leader: Option<String>,
    pub role: SlotRole,
    pub start_ms: u64,
    pub end_ms: u64,
}
```

Leader assignment: `leader_index = slot % validators.len()`. Deterministic given the same validator set and slot number.

## Monolith

Monoliths compress multiple blocks into a single structure for efficient archival and sync:

```rust
pub struct MonolithHeader {
    pub exec_height: u64,      // execution height
    pub window_start: u64,     // first block in monolith
    pub epoch_id: u64,         // epoch identifier
    pub block_count: u64,      // number of blocks included
    pub size_bytes: u64,       // total size
    pub monolith_id: String,   // unique identifier
    pub produced_at_ms: u64,   // production timestamp
    pub cosignatures: Vec<..>, // validator cosignatures
}
```

## Metrics

The metrics system provides Prometheus-compatible metrics:

```rust
pub struct MetricsProvider {
    // Register counters, gauges, histograms
    pub fn register_counter(name: &str, help: &str) -> Counter;
    pub fn register_gauge(name: &str, help: &str) -> Gauge;
}
```

Categories: blockchain, mempool, network, storage, execution, system, security, tokenomics.

Endpoint: `http://127.0.0.1:9090/metrics` (5-second update interval).

## Dual Core Structure

There are two `savitri-core` directories:

| Path | Purpose | Dependencies |
|------|---------|-------------|
| `Savitri-core/Savitri-core/` | Standalone (crates.io-ready) | Zero internal deps |
| `Savitri-core/` | Extended version | savitri-storage, savitri-zkp |

The standalone version is MIT licensed and suitable for external consumption. The extended version adds features like RocksDB integration, compression, and libp2p support.
