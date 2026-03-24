# Node Types

Savitri Network supports three node types, each optimized for different roles.

## Masternode

**Role**: Network coordinator, group formation, block acceptance certification.

```bash
./target/release/savitri-masternode config/masternode.toml
```

| Feature | Details |
|---------|---------|
| Consensus | Full BFT + PoU |
| Storage | RocksDB (required) |
| P2P | Full gossipsub mesh |
| Groups | Forms and manages lightnode groups |
| Block production | No (coordinates lightnodes) |
| RPC | Optional (`rpc` feature) |
| ZKP | Optional (`zkp-plonk`, `zkp-arkworks`) |

### Configuration

```toml
# config/masternode/mn-1.toml
[network]
listen_port = 5021
max_peers = 50
group_size = 7
group_node_timeout_secs = 600

[consensus]
block_time_secs = 5
slashing_enabled = true

[storage]
db_path = "data/mn1"
```

### Key Responsibilities

- **Group formation**: Assigns lightnodes to groups based on PoU scores
- **Block acceptance**: Verifies Block Acceptance Certificates (BAC) from groups
- **Peer management**: Tracks liveness via gossipsub messages, removes inactive nodes after timeout
- **Group announcements**: Publishes group assignments every ~30 seconds

## Lightnode

**Role**: Block producer, transaction processor, PoU participant.

```bash
cargo run -p savitri-lightnode --bin lightnode -- \
  --listen-port 4001 \
  --tx-interval-secs 2 \
  --block-interval-secs 10 \
  --max-block-txs 32 \
  --bootstrap PEER_ID@/ip4/127.0.0.1/tcp/5021
```

| Feature | Details |
|---------|---------|
| Consensus | PoU scoring + intra-group election |
| Storage | RocksDB (optional on mobile) |
| P2P | Gossipsub + direct request-response |
| Groups | Participates in masternode-assigned groups |
| Block production | Yes (when elected as proposer) |
| RPC | Yes (port 8545) |
| Mempool | Full SIMD-optimized pipeline |

### CLI Arguments

| Argument | Default | Description |
|----------|---------|-------------|
| `--listen-port` | 4001 | P2P listening port |
| `--tx-interval-secs` | 2 | Transaction generation interval (0 = max speed) |
| `--block-interval-secs` | 10 | Block production interval |
| `--max-block-txs` | 32 | Max transactions per block |
| `--db` | `lightnode.db` | Database path |
| `--network-key-path` | `lightnode-network.key` | P2P identity key |
| `--producer-key-path` | `lightnode-producer.key` | Block signing key |
| `--bootstrap` | None | Bootstrap peer (`PEER_ID@/ip4/IP/tcp/PORT`) |

### Feature Flags

| Flag | Description |
|------|-------------|
| `desktop` | Desktop optimizations (default) |
| `mobile` | Mobile optimizations (lighter storage) |
| `rocksdb` | RocksDB storage (default) |
| `lightweight-p2p` | Minimal P2P (fewer topics) |

## Guardian

**Role**: Archive/backup node, observer mode (no consensus participation).

```bash
./target/release/savitri-guardian
```

| Feature | Details |
|---------|---------|
| Consensus | None (observer only) |
| Storage | RocksDB (full archive) |
| P2P | Subscribe-only (no publishing) |
| Block production | No |
| Monitoring | Yes (archive + monitoring features) |

### Use Cases

- Full chain archive
- Data backup and recovery
- Block explorer data source
- Analytics and monitoring

## Running a Local Network

### 2 Masternodes + 10 Lightnodes

```bash
# Windows
scripts/run_all.bat

# Or manually
./target/release/savitri-masternode exucutables/configs/masternode/mn-1.toml &
./target/release/savitri-masternode exucutables/configs/masternode/mn-2.toml &

for i in $(seq 1 10); do
  cargo run -p savitri-lightnode --bin lightnode -- \
    --listen-port $((5000 + i)) \
    --db "lightnode${i}.db" \
    --bootstrap PEER_ID@/ip4/127.0.0.1/tcp/5021 &
done
```

### Ports (Local Test)

| Node | P2P Port | Notes |
|------|----------|-------|
| MN 1-5 | 5021-5025 | Masternodes |
| LN 1-10 | 5001-5010 | Lightnodes |
| TX Generator | 5029 | Transaction generator |

## Monitoring

```bash
# Check running nodes
scripts/check_status.bat

# Stop all
scripts/stop_all.bat

# Prometheus metrics
curl http://localhost:9090/metrics
```

## Docker

```bash
# Single masternode
cd savitri-masternode
docker build -t savitri-masternode .
docker run -p 4021:4021 -p 9090:9090 savitri-masternode

# Full testnet
cd savitri-testnet
docker-compose up -d
# Grafana: http://localhost:3000
```
