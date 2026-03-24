# Cryptography Specification

Complete specification of all cryptographic primitives used in the Savitri Network.

## Signature Scheme

| Property | Value |
|----------|-------|
| Algorithm | Ed25519 |
| Library | `ed25519-dalek` |
| Private key | 32 bytes |
| Public key | 32 bytes (compressed Edwards point) |
| Signature | 64 bytes |
| Address | Hex-encoded public key (64 hex characters) |

### Transaction Signing

```
message = from_hex_bytes || to_hex_bytes || amount_le_u64 || nonce_le_u64 || fee_le_u128
hash    = SHA-256(message)
sig     = Ed25519_sign(signing_key, hash)
```

### Block Signing

Blocks are signed by the proposer using Ed25519 over the block header hash (SHA-512).

### Peer Identity

libp2p peers use Ed25519 for their peer identity keys (separate from account keys).

## Hash Functions

### SHA-256

| Usage | Input | Output |
|-------|-------|--------|
| Transaction signing message | Canonical TX bytes | 32 bytes |
| Monolith commitment | Header fields | 32 bytes |
| ZKP statement hash | Statement fields | 32 bytes |
| AES key derivation | Password | 32 bytes |

### SHA-512

| Usage | Input | Output |
|-------|-------|--------|
| Block hash | `"BLK"` + version byte + header bytes | 64 bytes |
| Headers commit | Binary tree of block headers | 64 bytes |

### BLAKE3

| Usage | Input | Output |
|-------|-------|--------|
| Fast hashing | General purpose | Variable |
| State computation | State data | 32 bytes |

### Keccak256

| Usage | Input | Output |
|-------|-------|--------|
| Contract storage slots | `address \|\| base_slot` | 8 bytes (u64) |
| Nested mappings | `key \|\| inner_hash` | 8 bytes (u64) |

## Domain Separation

All hash computations include domain tags to prevent cross-domain collisions:

| Domain | Tag | Hash | Output |
|--------|-----|------|--------|
| Block | `"BLK"` + `version: u8` | SHA-512 | 64 bytes |
| TX root | `"TXv1"` leaf tag | Rolling accumulator | 32 bytes |
| State root | `"STATEv1-LE"` seed + `"STATE"` leaf | Lexicographic snapshot | 32 bytes |

### Transaction Root

Computed as a rolling accumulator over canonical bincode of transactions:

```
accumulator = H("TXv1")
for tx in transactions:
    leaf = bincode::serialize(tx)
    accumulator = H(accumulator || leaf)
tx_root = accumulator
```

### State Root

Computed from a lexicographic database snapshot:

```
seed = H("STATEv1-LE")
for (key, value) in storage.iter_sorted():
    leaf = H("STATE" || key || value)
    seed = H(seed || leaf)
state_root = seed
```

## Encryption

### AES-256-GCM (At Rest)

Used for:
- Private key encryption in the SDK
- Mobile database column encryption
- Device data encryption in IoT connector

| Property | Value |
|----------|-------|
| Algorithm | AES-256-GCM |
| Key size | 256 bits |
| Nonce size | 96 bits (12 bytes) |
| Tag size | 128 bits (16 bytes) |
| Key derivation | Password-based |

### P2P Encryption

| Property | Value |
|----------|-------|
| Protocol | Noise (XX handshake pattern) |
| Key exchange | X25519 |
| Cipher | ChaChaPoly |
| Multiplexing | Yamux |

## Key Derivation (Mobile)

### BIP-39

- **Wordlist**: English (2048 words)
- **Entropy**: 128 bits (12 words) or 256 bits (24 words)
- **Mnemonic → Seed**: PBKDF2-HMAC-SHA512 (2048 rounds)

### BIP-44

- **Path**: `m/44'/1337'/0'/0/0`
- **Coin type**: 1337 (Savitri)
- **Account**: 0
- **Change**: 0 (external)
- **Index**: 0

### Key Hierarchy

```
mnemonic → seed (512 bits)
    → master_key (via HMAC-SHA512)
        → m/44' → m/44'/1337' → m/44'/1337'/0'
            → m/44'/1337'/0'/0 → m/44'/1337'/0'/0/0
                → ed25519_private_key (32 bytes)
                    → ed25519_public_key (32 bytes)
                        → address (hex-encoded, 64 chars)
```

## Zero-Knowledge Proofs

See [ZKP Architecture](../architecture/zkp.md) for details.

| Backend | Curve | Proof System | Use Case |
|---------|-------|-------------|----------|
| Mock | N/A | Always-valid | Testing |
| Arkworks | BN254 | Groth16 | Production |
| PLONK | BN254 | PLONK | General-purpose |

## Security Properties

### Constant-Time Operations

All signature verification uses constant-time comparison (via `subtle` crate) to prevent timing side-channel attacks.

### Key Zeroization

Private keys are zeroized on drop using the `zeroize` crate in both the SDK and mobile app.

### Replay Protection

- **Transactions**: Nonce-based (monotonically increasing per account)
- **FL updates**: Per-round nonce per trainer
- **Oracle feeds**: Sequence number tracking
- **P2P messages**: Gossipsub deduplication
