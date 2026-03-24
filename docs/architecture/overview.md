# Architecture Overview

Savitri Network is structured as a monorepo with 13 git submodules. Each Rust crate is independent with local path dependencies.

## Crate Hierarchy

```
savitri-core (foundation: types, crypto, slot scheduler)
├── savitri-storage (RocksDB, column families, state roots)
│   ├── savitri-mempool (SIMD tx scoring, adaptive weights, LRU cache)
│   ├── savitri-contracts (smart contracts, governance/DAO, oracle, token standards)
│   └── savitri-rpc (axum HTTP API, JSON-RPC 2.0)
├── savitri-consensus (BFT voting, PoU scoring, group-aware)
├── savitri-p2p (libp2p 0.55: gossipsub, kademlia, noise, yamux)
└── savitri-zkp (mock/arkworks/plonk backends)

Node binaries:
    savitri-masternode  — full validator (all modules)
    savitri-lightnode   — desktop/mobile optimized
    savitri-guardian    — archive/backup observer

Applications:
    savitri-game-api    — TAP game HTTP backend
    savitri-mobile      — Flutter wallet/game app
    Savitri_installer   — Tauri desktop installer
```

## Crate Roles

| Crate | Role | Key Dependencies |
|-------|------|-----------------|
| `savitri-core` | Foundation types (Transaction, Account, Block), ed25519 crypto, slot scheduler, compression | ed25519-dalek, blake3, sha2, bincode |
| `savitri-storage` | RocksDB with column families (blocks, tx, accounts, receipts, meta). State root via lexicographic DB snapshot | rocksdb, bincode |
| `savitri-p2p` | Gossipsub on topics `savitri-block` and `savitri-tx`. Kademlia DHT. Compression (snap, lz4) | libp2p 0.55 |
| `savitri-consensus` | BFT with 2f+1 quorum (0.67). Group-aware, PoU-based, and lightweight modes | rayon, async-trait |
| `savitri-mempool` | SIMD-optimized tx processing (AVX2+FMA on x86_64, NEON on ARM). LRU score cache | rayon, crossbeam, dashmap |
| `savitri-zkp` | Multiple backends: `mock` (default), `plonk` (halo2), `arkworks` (ark-bn254/groth16) | ark-bn254, halo2 |
| `savitri-contracts` | Smart contract runtime, DAO governance, oracle, SAVITRI-20/721/1155 | rocksdb, rayon |
| `savitri-rpc` | Axum-based JSON-RPC 2.0 API for lightnode and masternode | axum, tower |
| `savitri-sdk` | Client library: RPC client, wallet, tx builders. CLI tools | reqwest, ed25519-dalek |

## Domain Separation

The protocol uses distinct domain tags to prevent cross-domain hash collisions:

| Domain | Tag | Hash |
|--------|-----|------|
| Block hash | `"BLK"` + version byte | SHA-512 |
| Transaction root | `"TXv1"` leaf tag | Rolling accumulator over canonical bincode |
| State root | `"STATEv1-LE"` seed + `"STATE"` leaf tag | Lexicographic DB snapshot |

## Data Flow

```
1. Transaction created (SDK/wallet)
       │
2. Signed with Ed25519 (SHA-256 message hash)
       │
3. Submitted via JSON-RPC → savitri_sendRawTransaction
       │
4. Mempool admission (nonce check, balance check, quota check)
       │
5. SIMD batch scoring (fee × 0.7 + class_priority × 0.3)
       │
6. Block proposer drains top-N transactions
       │
7. Block produced → gossipsub broadcast
       │
8. BFT voting (2f+1 quorum) → Block Acceptance Certificate
       │
9. Block committed to RocksDB
```

## Feature Flags

| Crate | Default Features | Notable Optional |
|-------|------------------|------------------|
| `savitri-core` | _(none)_ | `rocksdb`, `compression`, `libp2p`, `full`, `testnet` |
| `savitri-lightnode` | `desktop, rocksdb` | `mobile`, `lightweight-p2p` |
| `savitri-masternode` | `full` | `rpc`, `zkp-plonk`, `zkp-arkworks` |
| `savitri-mempool` | `simd, cache` | `adaptive_weights` |
| `savitri-zkp` | `mock` | `plonk`, `arkworks`, `production` |
| `savitri-consensus` | `std` | `group-aware`, `pou-based`, `bft`, `lightweight`, `full` |
| `savitri-contracts` | `governance, oracle` | `standards`, `erc20`, `fl` |
| `savitri-storage` | `rocksdb` | `memory`, `prometheus`, `tempfile` |
