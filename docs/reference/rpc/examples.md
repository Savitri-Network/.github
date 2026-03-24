# RPC Examples

Practical examples using `curl` and the Savitri SDK.

## curl Examples

### Health Check

```bash
curl -s -X POST http://localhost:8545/rpc \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"savitri_health","params":[],"id":1}'
```

### Get Block Height

```bash
curl -s -X POST http://localhost:8545/rpc \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"savitri_blockNumber","params":[],"id":1}'
```

### Get Block by Height

```bash
curl -s -X POST http://localhost:8545/rpc \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"savitri_getBlockByHeight","params":[42],"id":1}'
```

### Get Account

```bash
curl -s -X POST http://localhost:8545/rpc \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"savitri_getAccount","params":["aabbccdd...64hexchars"],"id":1}'
```

### Submit Transaction

```bash
curl -s -X POST http://localhost:8545/rpc \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"savitri_sendRawTransaction","params":["signed_tx_hex"],"id":1}'
```

### Faucet Claim

```bash
curl -s -X POST http://localhost:8545/rpc \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"savitri_faucetClaim","params":["your_address_hex"],"id":1}'
```

### PoU Status

```bash
# Local score
curl -s -X POST http://localhost:8545/rpc \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"savitri_pouLocal","params":[],"id":1}'

# All peers
curl -s -X POST http://localhost:8545/rpc \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"savitri_pouPeers","params":[],"id":1}'
```

### Batch Request

```bash
curl -s -X POST http://localhost:8545/rpc \
  -H "Content-Type: application/json" \
  -d '[
    {"jsonrpc":"2.0","method":"savitri_blockNumber","params":[],"id":1},
    {"jsonrpc":"2.0","method":"savitri_health","params":[],"id":2},
    {"jsonrpc":"2.0","method":"net_peerCount","params":[],"id":3}
  ]'
```

## SDK Examples

### Monitor Block Production

```rust
use savitri_sdk::RpcClient;
use std::time::Duration;

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    let client = RpcClient::from_url("http://localhost:8545")?;
    let mut last_height = 0u64;

    loop {
        let height = client.get_block_number().await?;
        if height > last_height {
            let block = client.get_block_by_height(height).await?;
            println!(
                "Block #{}: {} TXs, proposer: {}...{}",
                height,
                block.transaction_count,
                &block.proposer[..8],
                &block.proposer[56..]
            );
            last_height = height;
        }
        tokio::time::sleep(Duration::from_secs(1)).await;
    }
}
```

### Transfer with Balance Check

```rust
use savitri_sdk::{RpcClient, Wallet, TransactionBuilder};

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    let client = RpcClient::from_url("http://localhost:8545")?;
    let wallet = Wallet::from_private_key_hex("your_key")?;

    // Check balance first
    let account = client.get_account(wallet.address()).await?;
    let balance: u128 = account.balance.parse()?;
    let transfer_amount: u128 = 1_000_000_000_000_000_000; // 1 SAVT
    let fee: u128 = 1_000_000_000_000_000; // 0.001 SAVT

    if balance < transfer_amount + fee {
        println!("Insufficient balance: {} < {}", balance, transfer_amount + fee);
        return Ok(());
    }

    let tx = TransactionBuilder::new()
        .to("recipient_address_64hex")
        .value(transfer_amount)
        .nonce(account.nonce)
        .fee(fee)
        .build_and_sign(&wallet)?;

    // Serialize and send (simplified)
    println!("Transaction built, nonce: {}", account.nonce);

    Ok(())
}
```

### Multi-Node Health Dashboard

```rust
use savitri_sdk::RpcClient;

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    let nodes = vec![
        ("MN-1", "http://localhost:5021"),
        ("MN-2", "http://localhost:5022"),
        ("LN-1", "http://localhost:5001"),
        ("LN-2", "http://localhost:5002"),
    ];

    for (name, url) in &nodes {
        match RpcClient::from_url(*url) {
            Ok(client) => {
                match client.health().await {
                    Ok(h) => println!("{}: {} ({})", name, h.status, h.mode),
                    Err(e) => println!("{}: OFFLINE ({})", name, e),
                }
            }
            Err(e) => println!("{}: ERROR ({})", name, e),
        }
    }

    Ok(())
}
```

### Track PoU Scores

```rust
use savitri_sdk::RpcClient;

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    let client = RpcClient::from_url("http://localhost:8545")?;

    let pou = client.pou_local().await?;
    println!("My PoU score: {:?}", pou.local_score);
    println!("Leader: {:?}", pou.leader);
    println!("Am I leader: {}", pou.local_is_leader);

    let peers = client.pou_peers().await?;
    let mut scores: Vec<_> = peers.peers.iter().collect();
    scores.sort_by(|a, b| b.1.cmp(a.1));

    println!("\nPeer Rankings:");
    for (i, (peer, score)) in scores.iter().enumerate() {
        println!("  {}. {} (score: {})", i + 1, &peer[..16], score);
    }

    Ok(())
}
```
