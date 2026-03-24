# Installation

## System Requirements

### Rust (all node types)

- **Rust**: stable >= 1.82
- **Stack size**: 8MB (configured via linker flags)

### Linux

```bash
sudo apt install build-essential pkg-config clang llvm-dev libclang-dev \
  cmake ninja-build libssl-dev zlib1g-dev liblz4-dev libzstd-dev
```

### macOS

```bash
brew install rustup ccache cmake ninja llvm openssl zstd lz4
```

### Windows

- Visual Studio Build Tools with C++ workload
- Windows SDK
- Use the Developer Command Prompt or `build_win.bat`

## Building from Source

Savitri Network is a monorepo with independent crates (no root workspace `Cargo.toml`). Each crate is built individually.

### Build Node Binaries

```bash
# Masternode (full validator)
cargo build --release -p savitri-masternode

# Lightnode (desktop/mobile optimized)
cargo build --release -p savitri-lightnode

# Guardian (archive/observer)
cargo build --release -p savitri-guardian
```

### Build SDK Tools

```bash
cargo build --release -p savitri-sdk
```

This produces three CLI tools:
- `key_generator` -- Generate ed25519 keypairs
- `transaction_signer` -- Sign transactions offline
- `network_monitor` -- Monitor node health

### Build Game API

```bash
cargo build --release -p savitri-game-api
```

## Running Tests

```bash
# All tests
cargo test --workspace

# Specific crate
cargo test -p savitri-core
cargo test -p savitri-storage
cargo test -p savitri-consensus

# Single test by name
cargo test -p savitri-core -- test_name_here
```

## Linting

```bash
cargo fmt --all
cargo clippy --workspace --all-targets --all-features -- -D warnings
```

## Docker

A multi-stage Dockerfile is available for masternode:

```bash
cd savitri-masternode
docker build -t savitri-masternode .
```

Full testnet (Prometheus + Grafana + 2 masternodes + 1 lightnode + 1 guardian):

```bash
cd savitri-testnet
docker-compose up -d
```

## Next Steps

- [Quick Start](quickstart.md) -- Run your first node and send a transaction
- [Node Types](../node-operations/node-types.md) -- Choose the right node for your use case
- [SDK Overview](../sdk/overview.md) -- Build applications on Savitri
