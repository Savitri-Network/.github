# Light Client

The `LightClient` provides a simplified API for lightweight consumers that only need basic chain queries and transaction submission.

## Setup

```rust
use savitri_sdk::LightClient;

let client = LightClient::new("http://localhost:8545")?;
```

## API

```rust
// Connection check
let connected: bool = client.is_connected().await;

// Health
let health = client.health().await?;
println!("{} - {}", health.mode, health.status);

// Block height
let height: u64 = client.get_block_number().await?;

// Block by height
let block = client.get_block(42).await?;

// Account info
let account = client.get_account("address_hex").await?;
println!("Balance: {}, Nonce: {}", account.balance, account.nonce);

// Balance only
let balance: String = client.get_balance("address_hex").await?;

// Submit transaction
let result = client.send_raw_transaction("signed_tx_hex").await?;
println!("TX hash: {}", result.tx_hash);

// PoU status
let pou = client.pou_local().await?;
```

## When to Use LightClient vs RpcClient

| Feature | LightClient | RpcClient |
|---------|-------------|-----------|
| Health/ping | Yes | Yes |
| Block queries | Height + by-height | All block methods |
| Account queries | Balance + account | Balance, nonce, tokens |
| Transactions | Send raw | Send raw + get + receipt |
| PoU | Local only | Local, peers, groups, masternodes |
| Mempool | No | Size, status, pending |
| Tokens | No | Info, balance, transfers |
| Batch requests | No | Yes |
| Raw JSON-RPC | No | Yes (`call_raw`) |

Use `LightClient` for simple integrations. Use `RpcClient` directly when you need the full API surface. You can access the inner `RpcClient` via `client.rpc()` for advanced operations.
