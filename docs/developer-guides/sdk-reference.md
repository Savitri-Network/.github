# Savitri SDK

`savitri-sdk` is the official Rust client library for interacting with the Savitri Network. It provides:

- **RpcClient** -- JSON-RPC 2.0 client for all node APIs
- **LightClient** -- Simplified wrapper for common operations
- **Wallet** -- Ed25519 key management and transaction signing
- **ContractClient** -- High-level contract, oracle, and governance helpers
- **TransactionBuilder** -- Fluent builder for constructing transactions

## Installation

Add to your `Cargo.toml`:

```toml
[dependencies]
savitri-sdk = { path = "../savitri-sdk" }
tokio = { version = "1", features = ["full"] }
anyhow = "1"
```

For HTTP client support:

```toml
savitri-sdk = { path = "../savitri-sdk", features = ["http-client"] }
```

## Quick Start

```rust
use savitri_sdk::{RpcClient, Wallet, TransactionBuilder};

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    // Connect to a node
    let client = RpcClient::from_url("http://localhost:8545")?;

    // Health check
    let health = client.health().await?;
    println!("Node: {} ({})", health.service, health.mode);

    // Block height
    let height = client.get_block_number().await?;
    println!("Height: {}", height);

    // Create a wallet
    let wallet = Wallet::new();
    println!("Address: {}", wallet.address());

    // Check balance
    let account = client.get_account(wallet.address()).await?;
    println!("Balance: {} (nonce: {})", account.balance, account.nonce);

    Ok(())
}
```

## Modules

| Module | Description |
|--------|-------------|
| [RPC Client](rpc-client.md) | Full JSON-RPC 2.0 client |
| [Wallet](wallet.md) | Key generation, signing, balance queries |
| [Transactions](transactions.md) | Build, sign, and submit transactions |
| [Contracts](contracts.md) | Contract calls, oracle, governance |
| [Light Client](light-client.md) | Simplified API for lightweight consumers |

## Security

- **HTTPS enforcement**: Remote (non-localhost) endpoints require HTTPS by default. Set `allow_insecure: true` to override (not recommended).
- **TLS 1.2 minimum**: HTTPS connections enforce TLS 1.2+.
- **Key zeroization**: Private keys are zeroized on `Wallet::drop()` to prevent memory leaks.
- **Timeout**: Default 30-second request timeout.

## CLI Tools

The SDK includes three binary tools:

```bash
# Generate ed25519 keypairs
cargo run -p savitri-sdk --bin key_generator

# Sign transactions offline
cargo run -p savitri-sdk --bin transaction_signer

# Monitor node health
cargo run -p savitri-sdk --bin network_monitor
```
