# RPC Client

The `RpcClient` is the core transport layer for communicating with Savitri nodes via JSON-RPC 2.0.

## Creating a Client

```rust
use savitri_sdk::RpcClient;

// From URL (localhost HTTP allowed)
let client = RpcClient::from_url("http://localhost:8545")?;

// From URL (HTTPS required for remote)
let client = RpcClient::from_url("https://rpc.savitrinetwork.com")?;

// With custom configuration
use savitri_sdk::client::rpc_client::RpcClientConfig;

let client = RpcClient::new(RpcClientConfig {
    url: "http://localhost:8545".to_string(),
    timeout: Some(60),          // 60 second timeout
    allow_insecure: false,      // enforce HTTPS for remote
})?;
```

## Chain Queries

```rust
// Block height
let height: u64 = client.get_block_number().await?;

// Block by height
let block = client.get_block_by_height(42).await?;
println!("Block {} hash: {}", block.height, block.hash);
println!("Proposer: {}", block.proposer);
println!("TX count: {}", block.transaction_count);

// Block hash
let hash: String = client.get_block_hash(42).await?;
```

## Account Queries

```rust
// Full account info
let account = client.get_account("aabbcc...64chars").await?;
println!("Balance: {}", account.balance);  // u128 as decimal string
println!("Nonce: {}", account.nonce);

// Balance only
let balance: String = client.get_balance("aabbcc...").await?;

// Nonce only
let nonce: u64 = client.get_nonce("aabbcc...").await?;
```

## Transaction Operations

```rust
// Get transaction by hash
let tx = client.get_transaction("tx_hash_hex").await?;
println!("From: {} To: {} Amount: {}", tx.from, tx.to, tx.amount);

// Submit signed transaction
let result = client.send_raw_transaction("signed_tx_hex").await?;
println!("TX hash: {}", result.tx_hash);

// Claim testnet faucet
let claim = client.faucet_claim("your_address_64hex").await?;
println!("Received: {} (tx: {})", claim.amount, claim.tx_hash);
```

## Consensus Queries

```rust
// Local PoU state
let pou = client.pou_local().await?;
println!("My score: {:?}", pou.local_score);
println!("Leader: {:?} (score: {:?})", pou.leader, pou.leader_score);
println!("Am I leader? {}", pou.local_is_leader);

// All peer scores
let peers = client.pou_peers().await?;
for (peer_id, score) in &peers.peers {
    println!("{}: {}", peer_id, score);
}

// Groups (masternode only)
let groups = client.pou_groups().await?;
for group in &groups.groups {
    println!("Group {}: {} members", group.group_id, group.members.len());
}

// Masternodes (masternode only)
let mns = client.pou_masternodes().await?;
for mn in &mns.masternodes {
    println!("{}: PoU={} Health={}", mn.node_id, mn.pou_score, mn.health_score);
}
```

## Batch Requests

Send multiple RPC calls in a single HTTP request (max 100):

```rust
let responses = client.batch(vec![
    ("savitri_blockNumber", serde_json::json!([])),
    ("savitri_health", serde_json::json!([])),
    ("savitri_getAccount", serde_json::json!(["aabbcc..."])),
]).await?;

for resp in &responses {
    println!("{}", resp);
}
```

## Raw Calls

For methods not covered by typed helpers:

```rust
let result: serde_json::Value = client
    .call_raw("custom_method", serde_json::json!(["param1", 42]))
    .await?;
```

## Health Check

```rust
// Typed health check
let health = client.health().await?;
assert_eq!(health.status, "ok");

// Simple ping (returns bool)
let is_alive = client.ping().await?;
```

## Error Handling

```rust
use savitri_sdk::SdkError;

match client.get_account("invalid").await {
    Ok(account) => println!("Balance: {}", account.balance),
    Err(SdkError::RpcError { code, message, .. }) => {
        println!("RPC error {}: {}", code, message);
    }
    Err(SdkError::HttpError(e)) => {
        println!("Network error: {}", e);
    }
    Err(e) => println!("Other error: {}", e),
}
```

Error types:

| Error | Description |
|-------|-------------|
| `SdkError::RpcError` | JSON-RPC error from server (code, message, data) |
| `SdkError::HttpError` | Network/transport error |
| `SdkError::JsonError` | JSON serialization error |
| `SdkError::InvalidResponse` | Unexpected response format |
| `SdkError::NoRpcClient` | Wallet method called without RPC connection |
