# Quick Start

Get a local Savitri node running and send your first transaction in under 5 minutes.

## 1. Generate Keys

```bash
# Generate a node keypair
cargo run -p savitri-sdk --bin key_generator
```

This outputs an ed25519 keypair:
- **Private key**: 64 hex characters (32 bytes) -- keep this secret
- **Public key**: 64 hex characters (32 bytes) -- this is your address

## 2. Start a Lightnode

```bash
cargo run -p savitri-lightnode --bin lightnode -- \
  --listen-port 4001 \
  --tx-interval-secs 2 \
  --block-interval-secs 10 \
  --max-block-txs 32
```

The lightnode starts listening on port 4001 (P2P) and 8545 (RPC).

## 3. Check Node Health

```bash
curl -X POST http://localhost:8545/rpc \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "savitri_health",
    "params": [],
    "id": 1
  }'
```

Response:

```json
{
  "jsonrpc": "2.0",
  "result": {
    "status": "ok",
    "service": "savitri-rpc",
    "mode": "lightnode"
  },
  "id": 1
}
```

## 4. Get Block Height

```bash
curl -X POST http://localhost:8545/rpc \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "savitri_blockNumber",
    "params": [],
    "id": 2
  }'
```

## 5. Check Account Balance

```bash
curl -X POST http://localhost:8545/rpc \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "savitri_getAccount",
    "params": ["YOUR_ADDRESS_HEX_64_CHARS"],
    "id": 3
  }'
```

## 6. Claim Testnet Tokens (Faucet)

On testnet, claim free SAVT tokens:

```bash
curl -X POST http://localhost:8545/rpc \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "savitri_faucetClaim",
    "params": ["YOUR_ADDRESS_HEX_64_CHARS"],
    "id": 4
  }'
```

Response:

```json
{
  "jsonrpc": "2.0",
  "result": {
    "result": {
      "tx_hash": "0x...",
      "amount": "5000000000000000000"
    }
  },
  "id": 4
}
```

Faucet dispenses 5 SAVT per claim with a 24-hour cooldown.

## 7. Using the SDK (Rust)

Add to your `Cargo.toml`:

```toml
[dependencies]
savitri-sdk = { path = "../savitri-sdk" }
tokio = { version = "1", features = ["full"] }
```

```rust
use savitri_sdk::{RpcClient, Wallet, TransactionBuilder};

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    let client = RpcClient::from_url("http://localhost:8545")?;

    // Health check
    let health = client.health().await?;
    println!("Node mode: {}", health.mode);

    // Block height
    let height = client.get_block_number().await?;
    println!("Block height: {}", height);

    // Create wallet and check balance
    let wallet = Wallet::new();
    let account = client.get_account(wallet.address()).await?;
    println!("Address: {}", wallet.address());
    println!("Balance: {}", account.balance);

    Ok(())
}
```

## Next Steps

- [SDK Reference](../sdk/overview.md) -- Full SDK documentation
- [RPC API Reference](../rpc/overview.md) -- All JSON-RPC methods
- [Send a Transaction](../sdk/transactions.md) -- Build, sign, and send transactions
