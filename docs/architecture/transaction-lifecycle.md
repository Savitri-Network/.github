# Transaction Pipeline

The transaction pipeline handles the full lifecycle from creation to finalization.

## Overview

```
User/SDK
  │ build + sign (Ed25519)
  ▼
JSON-RPC (savitri_sendRawTransaction)
  │ hex decode → bytes_to_raw_tx
  ▼
Mempool Admission
  │ stateless guards → stateful guards
  ▼
SIMD Batch Scoring
  │ fee * 0.7 + class_priority * 0.3
  ▼
Block Proposer (drain_for_block_production)
  │ top-N by score, nonce-ordered per sender
  ▼
Block Production
  │ compute tx_root, state_root
  ▼
Gossipsub Broadcast (/savitri/block/1)
  │
  ▼
BFT Voting (2f+1 quorum)
  │ Block Acceptance Certificate
  ▼
RocksDB Commit
  │ CF_BLOCKS, CF_TRANSACTIONS, CF_ACCOUNTS
  ▼
Finalized
```

## Transaction Format

### Canonical Signing Message

```
from (hex bytes) || to (hex bytes) || amount (u64 LE) || nonce (u64 LE) || fee (u128 LE)
```

This message is SHA-256 hashed, then signed with Ed25519.

### Wire Format (raw_tx_hex)

```
from_hex_bytes || to_hex_bytes || amount_le_u64 || nonce_le_u64 || fee_le_u128 || pubkey (32B) || signature (64B) || data (optional)
```

Submitted as hex-encoded string to `savitri_sendRawTransaction`.

## Mempool Admission

### Stateless Guards (no storage access)

1. Signature format validation (64-byte ed25519)
2. Public key format validation (32-byte ed25519)
3. Signature verification against message hash
4. Transaction size limits

### Stateful Guards (storage access required)

1. **Nonce check**: `tx.nonce >= account.nonce` (gap tolerance up to 5000)
2. **Balance check**: `account.balance >= tx.amount + tx.fee`
3. **Quota check**: Per-class and per-sender limits

### Transaction Classes

| Class | Priority | Max Quota |
|-------|----------|-----------|
| FederatedUpdate | 1.0 (highest) | 50,000 |
| Financial | 0.8 | 50,000 |
| IoTData | 0.5 | 100,000 |
| Governance | 0.7 | 50,000 |

### Quota System

- **Per-class limits**: Financial 50K, IoT 100K, Global 100K
- **Per-sender cap**: 512 pending transactions per sender
- **Fair drain**: `drain_fair_batch()` respects quotas and calls `record_removal()`

## SIMD Scoring

The dispatcher scores transactions using SIMD-optimized batch computation:

```
score = fee_normalized * fee_weight + class_priority * class_weight
```

Default weights: `fee_weight = 0.7`, `class_weight = 0.3`.

### SIMD Implementation

| Architecture | Instruction Set | Lanes | Threshold |
|-------------|----------------|-------|-----------|
| x86_64 | AVX2 + FMA | 4 doubles | 32 TXs |
| ARM (aarch64) | NEON | 2 doubles | 32 TXs |
| Fallback | Scalar | 1 | Always |

For batches < 32 transactions, scalar computation is used (SIMD setup overhead exceeds benefit).

### Score Cache

LRU cache avoids recomputing scores for transactions that remain in the mempool across blocks.

### Adaptive Weights

With the `adaptive_weights` feature flag, weights adjust dynamically based on:
- Fee distribution in the mempool
- Class diversity
- Historical throughput

## Block Production

The elected proposer:

1. Calls `drain_for_block_production(max_block_txs)`
2. Gets top-N transactions ordered by score
3. Runs `final_validation` with `pending_nonces` tracking (prevents stale nonce rejection)
4. Computes transaction root: `"TXv1"` leaf tag, rolling accumulator
5. Computes state root: `"STATEv1-LE"` seed, lexicographic snapshot
6. Signs block with Ed25519
7. Broadcasts via gossipsub

### Nonce Handling

- `final_validation` tracks `pending_nonces` HashMap for drained-but-uncommitted TXs
- **Cold-start fix**: Accepts lowest available nonce when `storage_nonce=0 AND pending_nonces=0`
- **Nonce reset**: TX generator resets when local nonce > storage_nonce + 200

## Storage

Committed blocks are stored in RocksDB column families:

| Column Family | Key | Value |
|--------------|-----|-------|
| `CF_BLOCKS` | height (u64 LE) | Block (bincode) |
| `CF_TRANSACTIONS` | tx_hash | Transaction (bincode) |
| `CF_ACCOUNTS` | address (32 bytes) | Account (bincode) |
| `CF_METADATA` | `"chain_head"` | Latest block (bincode) |

Empty accounts (balance=0, nonce=0) are never persisted.
