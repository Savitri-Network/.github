# Tutorial: Deploy Your First SAVITRI-20 Token

This tutorial walks you through deploying a fungible token on the Savitri Network using the SDK.

## Prerequisites

- A running Savitri lightnode (see [Quick Start](../getting-started/quickstart.md))
- Rust toolchain installed
- A funded wallet (use the testnet faucet)

## 1. Project Setup

Create a new Rust project:

```bash
cargo new my-savitri-token
cd my-savitri-token
```

Add dependencies to `Cargo.toml`:

```toml
[dependencies]
savitri-sdk = { path = "../savitri-sdk" }
tokio = { version = "1", features = ["full"] }
anyhow = "1"
hex = "0.4"
```

## 2. Create a Wallet

```rust
use savitri_sdk::{Wallet, RpcClient};

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    // Create or import a wallet
    let wallet = Wallet::new();
    println!("Your address: {}", wallet.address());

    // Connect to local node
    let client = RpcClient::from_url("http://localhost:8545")?;

    // Check if node is running
    let health = client.health().await?;
    println!("Connected to {} ({})", health.service, health.mode);

    Ok(())
}
```

## 3. Fund Your Wallet

Claim testnet tokens via the faucet:

```rust
let claim = client.faucet_claim(wallet.address()).await?;
println!("Received {} SAVT (tx: {})", claim.amount, claim.tx_hash);

// Wait for confirmation
tokio::time::sleep(std::time::Duration::from_secs(10)).await;

// Verify balance
let account = client.get_account(wallet.address()).await?;
println!("Balance: {} (nonce: {})", account.balance, account.nonce);
```

## 4. Deploy the Token Contract

Token deployment is a transaction with `to = None` and the initialization data in `data`:

```rust
use savitri_sdk::TransactionBuilder;

// Encode token initialization parameters
fn encode_token_init(name: &str, symbol: &str, initial_supply: u128) -> Vec<u8> {
    let mut data = b"initialize_savitri20".to_vec();
    data.push(0); // separator

    // Name (length-prefixed)
    data.extend_from_slice(&(name.len() as u32).to_le_bytes());
    data.extend_from_slice(name.as_bytes());

    // Symbol (length-prefixed)
    data.extend_from_slice(&(symbol.len() as u32).to_le_bytes());
    data.extend_from_slice(symbol.as_bytes());

    // Initial supply (u128 LE)
    data.extend_from_slice(&initial_supply.to_le_bytes());

    data
}

// Build deploy transaction
let nonce = client.get_nonce(wallet.address()).await?;

let deploy_data = encode_token_init(
    "My Token",                              // name
    "MTK",                                   // symbol
    1_000_000_000_000_000_000_000_000,       // 1M tokens (18 decimals)
);

let deploy_tx = TransactionBuilder::new()
    // No .to() — indicates contract deployment
    .data(deploy_data)
    .value(0)
    .nonce(nonce)
    .fee(5_000_000_000_000_000) // 0.005 SAVT contract fee
    .build_and_sign(&wallet)?;

println!("Deploy TX built. Contract will be created at a derived address.");
```

## 5. Interact with the Token

Once deployed, use the `ContractClient` to interact:

```rust
use savitri_sdk::ContractClient;

let contract = ContractClient::from_url_and_wallet(
    "http://localhost:8545",
    wallet.clone(),
)?;

// Transfer tokens
let tx_hash = contract.call_contract(
    &token_contract_address,
    b"transfer",
    &encode_transfer(&recipient_address, 1000_000_000_000_000_000), // 1000 tokens
    Some(0),
).await?;
println!("Transfer TX: {}", tx_hash);
```

## 6. Check Token Balance

```rust
// Via RPC
let balance = client.call_raw(
    "account_getTokenBalance",
    serde_json::json!([wallet.address(), token_contract_address]),
).await?;
println!("Token balance: {}", balance);
```

## Token Standards Quick Reference

| Standard | Use Case | Key Functions |
|----------|----------|---------------|
| [SAVITRI-20](../smart-contracts/savitri-20.md) | Fungible tokens | `transfer`, `approve`, `transferFrom` |
| [SAVITRI-721](../smart-contracts/savitri-721.md) | NFTs | `mint`, `transferFrom`, `tokenURI` |
| [SAVITRI-1155](../smart-contracts/savitri-1155.md) | Multi-asset | `safeTransferFrom`, `balanceOfBatch` |

## Fee Reference

| Operation | Fee (SAVT) |
|-----------|-----------|
| Token deploy | 0.005 |
| Token transfer | 0.005 |
| Token approve | 0.005 |
| NFT mint | 0.005 |
| Batch transfer | 0.005 |

## Amounts and Decimals

All Savitri tokens use **18 decimals**:

```
1 token     = 1_000_000_000_000_000_000  (10^18)
0.001 token = 1_000_000_000_000_000      (10^15)
1M tokens   = 1_000_000_000_000_000_000_000_000  (10^24)
```

## Next Steps

- [SAVITRI-20 Reference](../smart-contracts/savitri-20.md) -- Full fungible token API
- [Contract Runtime](../smart-contracts/runtime.md) -- How contracts execute
- [Governance](../governance/overview.md) -- Propose token parameter changes
