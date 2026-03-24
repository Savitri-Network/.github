# Fault Tolerance and Finality

## BFT Guarantees

Savitri consensus uses a BFT (Byzantine Fault Tolerant) voting protocol with a **2f+1 quorum** (67% threshold).

### Fault Tolerance

| Total Nodes (n) | Max Byzantine (f) | Quorum Required |
|-----------------|-------------------|-----------------|
| 3 | 0 | 2 |
| 5 | 1 | 4 |
| 7 | 2 | 5 |
| 10 | 3 | 7 |
| 15 | 4 | 11 |

Formula: `f = floor((n-1) / 3)`, quorum = `2f + 1`

### Safety

A block is only finalized when a **Block Acceptance Certificate** (BAC) is produced with signatures from 2f+1 validators. This guarantees:

- **No conflicting blocks** can be finalized at the same height
- **No rollbacks** once a BAC is issued (deterministic finality, not probabilistic)
- **Byzantine nodes cannot** forge certificates without quorum participation

### Liveness

The network can produce blocks as long as `n - f` honest nodes are online:

- **Proposer rotation**: After 50 blocks, proposer steps down to prevent single-point failure
- **Election watchdog**: Re-triggers election if no block progress for 60 seconds
- **Progressive quorum relaxation**: After 3+ consecutive election failures, minimum candidates drops to 2

## DAG Structure

BlockHeaders support multi-parent DAG structure for concurrent block production:

```rust
pub struct BlockHeader {
    pub parent_hash: Vec<u8>,         // primary parent (backward compatible)
    pub parent_hashes: Vec<Vec<u8>>,  // additional parents for DAG
    // ...
}
```

### Conflict Detection

The `ConflictDetector` identifies:
- **Double-spend conflicts**: Same UTXO referenced in multiple branches
- **State conflicts**: Conflicting state transitions
- **Fork detection**: Multiple blocks at the same height from different proposers

### Conflict Resolution

When conflicts are detected:
1. Choose branch with higher cumulative PoU score
2. Revert conflicting transactions
3. Re-apply non-conflicting transactions from the losing branch

## Group-Aware Consensus

### Group Formation

Masternodes organize lightnodes into groups:

1. Sort lightnodes by PoU score
2. Assign to groups of `group_size` (3 dev, 7 prod)
3. Publish group assignments via gossipsub
4. Deterministic group IDs based on epoch (not wall-clock time)

### Intra-Group Election

Within each group:
1. Members share PoU scores
2. Highest PoU score becomes proposer (peer_id tiebreaker)
3. Minimum `(total+1)/2` candidates required
4. After 3+ failures: relax to minimum 2 candidates

### Block Acceptance

1. Group proposer produces a block
2. Group members verify and sign
3. BAC produced with 2f+1 signatures
4. Masternodes verify BAC and finalize

## Recovery Mechanisms

### Election Recovery

| Mechanism | Trigger | Action |
|-----------|---------|--------|
| Watchdog timer | No block for 60s | Re-trigger PoU + election |
| Progressive relaxation | 3+ failures | Lower quorum to 2 |
| Proposer rotation | 50 blocks | Step down, re-elect |
| Immediate re-election | Group re-init | Spawn PoU + election task |

### Peer Recovery

| Mechanism | Trigger | Action |
|-----------|---------|--------|
| Gossipsub keepalive | 60s interval | PoU score publish |
| Mesh recovery watchdog | No mesh for 60s | Re-dial group peers |
| SlowPeer disconnect | Queue saturated | Disconnect and reconnect |
| Nonce reset | Local > storage + 200 | Reset to storage nonce |

### Group Recovery

When a group is re-initialized:
1. PoU scores retained for continuing members
2. New members start with default scores
3. `intra_group_started` flag reset
4. Immediate PoU + election spawned (don't wait for periodic timers)

## Slashing (Production)

| Offense | Penalty | Cooldown |
|---------|---------|----------|
| Double vote | 50% stake | N/A |
| Downtime | 10% stake | Gradual |
| Invalid proposal | 25% stake | N/A |

Slashing is **disabled** on devnet, **enabled** on mainnet (`config/production.toml`).

## Consensus Configuration

| Parameter | Development | Production |
|-----------|-------------|------------|
| Timeout | 15s | 10s |
| Max rounds | 5 | 10 |
| Quorum threshold | 0.67 | 0.67 |
| BFT optimizations | Disabled | Enabled |
| Slashing | Disabled | Enabled |
| Block time | 2s | 5s |
| Group size | 3 | 7 |
