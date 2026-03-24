# Configuration Reference

All configuration files are in the `config/` directory. TOML format.

## Network

```toml
[network]
name = "devnet"              # "devnet", "testnet", "mainnet"
port = 8333                  # P2P listening port
rpc_port = 8545              # JSON-RPC port
ws_port = 8546               # WebSocket port
bootstrap_nodes = []         # Initial peers for discovery
max_peers = 50               # Maximum peer connections
connection_timeout = 30      # Timeout in seconds
```

| Parameter | Dev | Prod | Description |
|-----------|-----|------|-------------|
| `max_peers` | 10 | 50 | Max P2P connections |
| `connection_timeout` | 10s | 30s | Peer connection timeout |

## Storage

```toml
[storage]
data_dir = "./data"          # Database directory
cache_size_mb = 1024         # RocksDB block cache
write_buffer_size_mb = 64    # Write buffer per CF
max_open_files = 1000        # Max file descriptors
sync_interval = 30           # WAL sync interval (seconds)
compression = true           # Enable LZ4 compression
```

| Parameter | Dev | Prod | Description |
|-----------|-----|------|-------------|
| `cache_size_mb` | 256 | 1024 | Block cache size |
| `write_buffer_size_mb` | 16 | 64 | Write buffer |
| `compression` | false | true | Storage compression |

## Proof-of-Unity (PoU)

```toml
[pou]
scoring_update_interval = 300     # Score update (seconds)
group_formation_interval = 600    # Group rebalance (seconds)
consensus_sync_interval = 30      # Sync interval (seconds)
min_eligibility_score = 1000      # Min score (basis points, 0-10000)
min_stake_amount = 150            # Min stake for participation
min_uptime_requirement = 0.8      # Min uptime (0.0-1.0)

[pou.weights]
availability = 0.25               # Uptime weight
latency = 0.20                    # Response time weight
integrity = 0.20                  # Correctness weight
reputation = 0.20                 # Historical weight
participation = 0.15              # Activity weight

[pou.groups]
group_size = 7                    # Lightnodes per group
num_groups = 10                   # Number of groups
algorithm = "TopScore"            # Group formation algorithm
```

| Parameter | Dev | Prod | Description |
|-----------|-----|------|-------------|
| `scoring_update_interval` | 30s | 300s | How often PoU scores update |
| `group_formation_interval` | 60s | 600s | Group rebalance frequency |
| `min_eligibility_score` | 100 | 1000 | Min score to participate |
| `group_size` | 3 | 7 | Lightnodes per group |

## Performance

```toml
[performance]
max_scoring_time_ms = 1000        # Max scoring computation time
max_group_formation_time_ms = 3000
max_consensus_sync_time_ms = 500
target_tps = 1000                 # Target transactions per second
max_block_size = 2097152          # Max block size (bytes)
block_time_ms = 5000              # Target block time
```

| Parameter | Dev | Prod | Description |
|-----------|-----|------|-------------|
| `target_tps` | 100 | 1000 | Target throughput |
| `block_time_ms` | 2000 | 5000 | Block interval |

## Adaptive Latency

```toml
[adaptive_latency]
enabled = true
base_latency_ms = 100             # Baseline latency
max_latency_ms = 2000             # Max acceptable latency
adjustment_factor = 0.05          # Smoothing factor
epoch_window = 100                # Epochs for averaging
network_averaging = true          # Average across network
```

## Consensus

```toml
[consensus]
timeout = 10                      # Round timeout (seconds)
max_rounds = 10                   # Max voting rounds
quorum_threshold = 0.67           # BFT quorum (2f+1)
enable_bft_optimizations = true   # Fast-path for unanimous votes
```

## Slashing

```toml
[slashing]
enabled = true                    # Enable slashing (prod only)
double_vote_slash = 0.50          # 50% slash for double voting
downtime_slash = 0.10             # 10% slash for downtime
invalid_proposal_slash = 0.25     # 25% slash for invalid blocks
downtime_threshold = 0.2          # 20% missed blocks triggers slash
grace_period = 3600               # 1 hour grace before downtime slash
```

## Monitoring

```toml
[monitoring]
enabled = true
host = "127.0.0.1"
port = 9090
update_interval_secs = 5
log_level = "info"                # "debug", "info", "warn", "error"

[monitoring.thresholds]
block_stall_timeout_secs = 300
high_block_time_secs = 15
low_peer_count = 3
high_memory_usage = 0.9
```

## Score Cache

```toml
[cache]
max_entries = 10000               # LRU cache size
ttl_seconds = 300                 # Cache entry TTL
cleanup_interval = 60             # Cleanup frequency (seconds)
```

## Masternode Configs

Per-masternode configs in `exucutables/configs/masternode/mn-*.toml`:

```toml
[node]
listen_port = 5021
group_node_timeout_secs = 600     # Peer inactivity timeout

[storage]
db_path = "data/mn1"
```

## Environment Variables

| Variable | Description |
|----------|-------------|
| `RUST_LOG` | Log filter (e.g., `info,libp2p_gossipsub::behaviour=error`) |
| `CARGO_TARGET_DIR` | Build output directory |
| `RUSTFLAGS` | Compiler flags (e.g., `-C link-arg=/STACK:8388608`) |
