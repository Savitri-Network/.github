# Building and Sending Transactions

The `TransactionBuilder` provides a fluent API for constructing, signing, and submitting transactions to the Savitri Network.

## Simple Transfer

```rust
use savitri_sdk::{Wallet, TransactionBuilder, RpcClient};

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    let client = RpcClient::from_url("http://localhost:8545")?;
    let wallet = Wallet::from_private_key_hex("your_private_key_hex")?;

    // Get current nonce
    let nonce = client.get_nonce(wallet.address()).await?;

    // Build and sign
    let signed_tx = TransactionBuilder::new()
        .to("recipient_address_64hex")
        .value(1_000_000_000_000_000_000)  // 1 SAVT (18 decimals)
        .nonce(nonce)
        .fee(1_000_000_000_000_000)        // 0.001 SAVT fee
        .build_and_sign(&wallet)?;

    // Submit
    let result = client.send_raw_transaction(&serialize_tx(&signed_tx)).await?;
    println!("TX hash: {}", result.tx_hash);

    Ok(())
}
```

## Builder API

### Required Fields

| Method | Type | Description |
|--------|------|-------------|
| `.from(address)` | String | Sender address (auto-filled from wallet if omitted) |
| `.to(address)` | String | Recipient address |
| `.value(amount)` | u128 | Transfer amount in smallest unit |

### Optional Fields

| Method | Type | Default | Description |
|--------|------|---------|-------------|
| `.nonce(n)` | u64 | 0 | Transaction nonce (get from `get_nonce()`) |
| `.fee(f)` | u128 | None | Transaction fee |
| `.data(bytes)` | Vec\<u8\> | None | Payload for contract calls |

### Build Without Signing

```rust
let unsigned = TransactionBuilder::new()
    .from("sender_address")
    .to("recipient_address")
    .value(1000)
    .nonce(5)
    .build()?;

// unsigned.from, unsigned.to, unsigned.value, unsigned.nonce, unsigned.fee, unsigned.data
```

### Build and Sign

```rust
let signed = TransactionBuilder::new()
    .to("recipient_address")
    .value(1000)
    .nonce(5)
    .fee(100)
    .build_and_sign(&wallet)?;

// signed.transaction -- the UnsignedTransaction
// signed.public_key  -- 32-byte ed25519 public key
// signed.signature   -- 64-byte ed25519 signature
```

If `.from()` is not called, the wallet's address is used automatically.

## Oracle Calls

```rust
let signed = TransactionBuilder::new()
    .oracle_call(
        "oracle_contract_address",
        "request_data",
        b"temperature_sensor_01",
    )
    .value(0)
    .nonce(nonce)
    .build_and_sign(&wallet)?;
```

## Governance Calls

### Vote on a Proposal

```rust
use savitri_sdk::GovernanceAction;

let signed = TransactionBuilder::new()
    .governance_call(
        "governance_contract_address",
        proposal_id,              // u64
        GovernanceAction::Vote(true),  // true = support, false = oppose
    )
    .nonce(nonce)
    .build_and_sign(&wallet)?;
```

### Execute an Approved Proposal

```rust
let signed = TransactionBuilder::new()
    .governance_call(
        "governance_contract_address",
        proposal_id,
        GovernanceAction::Execute,
    )
    .nonce(nonce)
    .build_and_sign(&wallet)?;
```

### Create an FL Proposal

```rust
let signed = TransactionBuilder::new()
    .create_fl_proposal(
        "governance_contract_address",
        "Upgrade model v2",        // title
        "Replace BERT with GPT-4", // description
        604800,                    // voting period in seconds (7 days)
    )
    .nonce(nonce)
    .build_and_sign(&wallet)?;
```

## Transaction Signing Format

The signing message is constructed as:

```
from_hex_bytes || to_hex_bytes || amount_le_u64 || nonce_le_u64 || fee_le_u128
```

Then SHA-256 hashed, and the hash is signed with Ed25519.

This format matches the server-side verification in `savitri-rpc`.

## Amounts and Decimals

All amounts use 18 decimal places:

| Human Amount | Raw Value |
|-------------|-----------|
| 1 SAVT | `1_000_000_000_000_000_000` |
| 0.1 SAVT | `100_000_000_000_000_000` |
| 0.001 SAVT | `1_000_000_000_000_000` |
| 0.00001 SAVT | `10_000_000_000_000` |

## Fee Reference

| Transaction Type | Fee (SAVT) | Raw Value |
|-----------------|------------|-----------|
| Normal transfer | 0.001 | `1_000_000_000_000_000` |
| Contract call | 0.005 | `5_000_000_000_000_000` |
| IoT data | 0.00005 | `50_000_000_000_000` |
