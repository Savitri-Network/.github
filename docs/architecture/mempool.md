# Mempool Architecture

The mempool manages pending transactions from admission through scoring to block production draining.

## Pipeline

```
Network (gossipsub /savitri/tx/1)
    │
    ▼
Prevalidation (stateless)
    │ signature format, size limits
    ▼
Admission Control
    │ quota check, per-sender cap
    ▼
Stateful Validation
    │ nonce, balance, fee limits
    ▼
SIMD Batch Scoring
    │ fee * 0.7 + class * 0.3
    ▼
Score Cache (LRU)
    │
    ▼
Pending Pool
    │
    ▼
drain_for_block_production()
    │ top-N by score, final_validation
    ▼
Block Proposer
```

## Transaction Classes

| Class | Priority | Use Case |
|-------|----------|----------|
| `FederatedUpdate` | 1.0 (highest) | FL model updates |
| `Financial` | 0.8 | Token transfers |
| `Governance` | 0.7 | Votes, proposals |
| `IoTData` | 0.5 | Sensor data |

## Admission Quotas

| Class | Max Pending | Per-Sender Cap |
|-------|-------------|----------------|
| Financial | 50,000 | 512 |
| IoT | 100,000 | 512 |
| Global | 100,000 | 512 |

The `drain_fair_batch()` function calls `record_removal()` to properly decrement class counts when transactions are drained (fixed in Round 6).

## SIMD Scoring

### Score Formula

```
score = fee_normalized * fee_weight + class_priority * class_weight
```

Default: `fee_weight = 0.7`, `class_weight = 0.3`.

### Architecture-Specific Implementation

| Architecture | Intrinsics | Lanes | Min Batch |
|-------------|-----------|-------|-----------|
| x86_64 (AVX2+FMA) | `_mm256_set1_pd`, `_mm256_fmadd_pd` | 4 doubles | 32 |
| ARM (NEON) | `vdupq_n_f64`, `vfmaq_f64` | 2 doubles | 32 |
| Fallback | Scalar loop | 1 | Always |

For batches < 32 transactions, scalar computation is used because SIMD setup overhead exceeds the benefit.

### Adaptive Weights (Optional)

With `adaptive_weights` feature flag, weights adjust based on:

- **Fee distribution**: If fees are clustered high, increase fee_weight
- **Class diversity**: If many different classes, increase class_weight
- **Historical throughput**: Adjust based on recent blocks

Parameters:
```
base_fee_weight = 0.7
base_class_weight = 0.3
adaptation_rate = 0.1 (smoothing)
fee_threshold_high = 2,000,000,000
fee_threshold_low = 500,000,000
class_diversity_threshold = 0.5
```

## Score Cache

LRU cache avoids recomputing scores:

| Parameter | Development | Production |
|-----------|-------------|------------|
| Cache size | 100 entries | 10,000 entries |
| TTL | 60 seconds | 300 seconds |

## Nonce Management

### Standard Flow

1. TX arrives with nonce N
2. Check: `N >= account.storage_nonce`
3. Track in `pending_nonces` HashMap during draining
4. Commit nonce after block finalization

### Gap Tolerance

Nonce gap up to **5000** is accepted for burst recovery after prolonged stalls (increased from 1000 in Round 11).

### Cold-Start Fix

When `storage_nonce=0 AND pending_nonces=0`, the lowest available nonce is accepted as the starting point. This handles the case where TXs were drained but the block was never committed by a previous proposer.

### Nonce Reset

TX generator resets local nonce when `local_nonce > storage_nonce + 200` (`MAX_NONCE_AHEAD`). Previously local nonces could drift to 2000+ while storage was at 84, causing all TXs to be rejected.

## Replay Prevention

Transaction hashes are tracked to prevent replay attacks. Once a TX hash is seen, duplicate submissions are rejected.

## Fee Validation

```rust
FeeLimits {
    min_fee: 100_000_000_000_000,       // 0.0001 SAVT
    max_fee: 1_000_000_000_000_000_000, // 1.0 SAVT
}
```

## Feature Flags

| Flag | Description |
|------|-------------|
| `simd` | SIMD-optimized scoring (default) |
| `cache` | Score cache (default) |
| `adaptive_weights` | Dynamic weight adjustment |
