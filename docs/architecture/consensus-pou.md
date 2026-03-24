# Proof-of-Unity (PoU) Consensus

Proof-of-Unity is Savitri Network's consensus mechanism. It combines reputation-based scoring with BFT-style voting to achieve fast finality without the energy waste of Proof-of-Work or the capital lockup of Proof-of-Stake.

## Score Formula

Each node maintains a PoU score updated every epoch:

```
S_i(t) = alpha * S_i(t-1) + (1 - alpha) * (weighted components)
```

Where `alpha = 0.3-0.5` (smoothing factor for temporal stability).

### Score Components

| Component | Weight | Description |
|-----------|--------|-------------|
| Availability | 0.25-0.30 | Uptime and responsiveness |
| Latency | 0.10-0.20 | Network response time |
| Integrity | 0.20-0.25 | Correct behavior, no byzantine faults |
| Reputation | 0.20 | Historical participation and reliability |
| Participation | 0.15 | Active block production and validation |

### Final Score

```
PoU_i = round(S_i(t) * 1000)
```

Produces a score on a 0-1000 integer scale. The node with the highest PoU score becomes the block proposer.

## Election Process

1. **PoU score sharing**: Nodes periodically share their PoU scores via gossipsub
2. **Candidate collection**: Each node collects scores from group members
3. **Proposer determination**: Node with highest `pou_score` wins (peer_id tiebreaker)
4. **Minimum quorum**: Requires `(total+1)/2` candidates to start election
5. **Progressive relaxation**: After 3+ consecutive failures, minimum candidates drops to 2

### Election Recovery

- **Watchdog timer**: Re-triggers election if no block progress for 60 seconds
- **Proposer rotation**: After 50 blocks, proposer voluntarily steps down
- **Immediate re-election**: After group re-initialization, spawns PoU + election task immediately

## Group Formation

Masternodes organize lightnodes into groups for parallel block production:

```
Masternode (coordinator)
    ├── Group A: [LN1, LN2, LN3, LN4, LN5]
    ├── Group B: [LN6, LN7, LN8, LN9, LN10]
    └── ...
```

- **Group size**: 3-7 lightnodes (configurable)
- **Group ID**: Deterministic based on epoch (all masternodes produce the same ID)
- **Announcements**: Masternodes publish group assignments every ~30 seconds
- **Re-initialization**: When group membership changes, scores are retained for continuing members

## BFT Voting

Block finality requires a **2f+1 quorum** (67% of validators):

1. Proposer produces a block
2. Block is broadcast via gossipsub (`/savitri/block/1`)
3. Validators verify and vote
4. **Block Acceptance Certificate** (BAC) is produced with 2f+1 signatures
5. Masternodes verify the BAC and finalize the block

## Consensus Modes

| Mode | Feature Flag | Use Case |
|------|-------------|----------|
| `group-aware` | `group-aware` | Full group-based consensus |
| `pou-based` | `pou-based` | PoU scoring without groups |
| `bft` | `bft` | BFT voting only |
| `lightweight` | `lightweight` | Minimal consensus for mobile/embedded |
| `full` | `full` | All consensus features |

## PoU Tiers and Rewards

| Tier | Min Score | Reward Multiplier |
|------|-----------|-------------------|
| Bronze | 300 | 1.0x |
| Silver | 500 | 1.5x |
| Gold | 700 | 2.0x |
| Platinum | 900 | 3.0x |

Nodes below score 300 do not receive rewards.

## Querying PoU State

### Via RPC

```bash
# Local PoU state
curl -X POST http://localhost:8545/rpc \
  -d '{"jsonrpc":"2.0","method":"savitri_pouLocal","params":[],"id":1}'

# Peer scores
curl -X POST http://localhost:8545/rpc \
  -d '{"jsonrpc":"2.0","method":"savitri_pouPeers","params":[],"id":1}'
```

### Via SDK

```rust
let client = RpcClient::from_url("http://localhost:8545")?;

let pou = client.pou_local().await?;
println!("Score: {:?}, Leader: {:?}", pou.local_score, pou.leader);

let peers = client.pou_peers().await?;
for (peer, score) in &peers.peers {
    println!("{}: {}", peer, score);
}
```

## Network Topology

```
P2P Topics:
  /savitri/tx/1                    -- Transaction broadcast
  /savitri/block/1                 -- Block broadcast
  /savitri/consensus/cert/1        -- Block acceptance certificates
  /savitri/peer_registry/1         -- Peer keepalive
  /savitri/lightnode/group/announce/1  -- Group assignments
  /savitri/group/{id}/election     -- Per-group elections
```

## Configuration

Key consensus parameters (from `config/production.toml`):

| Parameter | Dev | Production |
|-----------|-----|------------|
| Block time | 2s | 5s |
| Max TPS | 100 | 1000 |
| Group size | 3 | 7 |
| Node timeout | 300s | 600s |
| Election interval | 300s | 300s |
| PoU sharing interval | 60s | 60s |
| Latency probe interval | 30s | 30s |
