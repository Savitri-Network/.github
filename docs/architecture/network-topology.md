# P2P Networking

Savitri Network uses **libp2p 0.55** for all peer-to-peer communication.

## Transport Stack

| Layer | Technology |
|-------|-----------|
| Transport | TCP |
| Encryption | Noise protocol |
| Multiplexing | Yamux |
| Discovery | Kademlia DHT |
| Pub/Sub | Gossipsub |
| Identity | Ed25519 peer keys |

## Gossipsub Topics

| Topic | Publisher | Subscriber | Purpose |
|-------|----------|------------|---------|
| `/savitri/tx/1` | Lightnodes | Lightnodes, Masternodes | Transaction broadcast |
| `/savitri/block/1` | Lightnodes | All nodes | Block broadcast |
| `/savitri/consensus/cert/1` | Lightnodes | Masternodes, Observer | Block acceptance certificates |
| `/savitri/peer_registry/1` | All nodes | Masternodes | Peer keepalive / registration |
| `/savitri/lightnode/group/announce/1` | Masternodes | Lightnodes | Group assignments |
| `/savitri/group/{id}/election` | Group members | Group members | Per-group elections |

## Gossipsub Configuration

| Parameter | Lightnode | Masternode |
|-----------|-----------|------------|
| `mesh_n` | 8 | 8 |
| `mesh_n_high` | 12 | 12 |
| `mesh_n_low` | 4 | 4 |
| `mesh_outbound_min` | 3 | 3 |
| `connection_handler_queue_len` | 50,000 | 50,000 |
| `max_transmit_size` | 256 KB | 256 KB |
| `flood_publish` | false | true (all subscribers) |

## Message Compression

| Format | Status | Use Case |
|--------|--------|----------|
| snap | Enabled | General P2P messages |
| lz4 | Enabled | High-throughput paths |
| zstd | Disabled on MSVC | Not used on Windows |

## Peer Liveness

### Lightnode

- **PoU score sharing**: Publishes via gossipsub every 60 seconds
- **Latency probes**: Every 30 seconds via direct P2P
- **Heartbeat**: Gossipsub publish on `/savitri/peer_registry/1`

### Masternode

- **`last_seen` refresh**: Updated on ANY gossipsub message from a peer
- **Inactivity timeout**: 600 seconds (configurable per MN config)
- **`cleanup_inactive()`**: Removes nodes that exceed timeout
- **SlowPeer handler**: Disconnects peers with saturated send queues

### Observer

- **30-second heartbeat**: Publishes on `/savitri/peer_registry/1`
- **Subscribes to**: `/savitri/consensus/cert/1` and `/savitri/block/1`

## Bootstrap

Bootstrap peers are configured in `config/bootstrap_nodes.json`:

```json
[
  {
    "peer_id": "12D3KooW...",
    "address": "/ip4/3.120.x.x/tcp/4001"
  }
]
```

Nodes connect to bootstrap peers on startup and discover additional peers via Kademlia DHT.

## Direct P2P (Request-Response)

Some operations use direct peer-to-peer communication instead of gossipsub:

| Operation | Protocol |
|-----------|----------|
| Latency probes | Direct request-response |
| PoU ACK | Point-to-point response |
| Election candidates | Direct P2P |

This avoids O(N^2) broadcast overhead for operations that only need to reach specific peers.

## Network Ports

| Port | Protocol | Service |
|------|----------|---------|
| 4001 | TCP | P2P (libp2p) default |
| 8333 | TCP | P2P (alternative) |
| 8545 | HTTP | RPC (JSON-RPC 2.0) |
| 8546 | WS | WebSocket |
| 9090 | HTTP | Prometheus metrics |

## Configuration

### `config/development.toml`

```toml
[network]
max_peers = 10
block_time_secs = 2
max_tps = 100
group_size = 3
```

### `config/production.toml`

```toml
[network]
max_peers = 50
block_time_secs = 5
max_tps = 1000
group_size = 7
slashing_enabled = true
```

## Log Filtering

To reduce verbose gossipsub logs:

```
RUST_LOG=info,libp2p_gossipsub::behaviour=error
```

This suppresses Send Queue warnings that include full message payloads (1-2KB each, causing 120MB+ logs).
