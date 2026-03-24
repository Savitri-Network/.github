# SAVITRI-20: Fungible Token Standard

SAVITRI-20 is the fungible token standard for the Savitri Network, equivalent to Ethereum's ERC-20. It enables creation and management of fungible tokens with transfer, approval, and allowance mechanics.

## Interface

| Function | Params | Returns | Description |
|----------|--------|---------|-------------|
| `initialize` | owner, name, symbol, initial_supply | - | Deploy and initialize a new token |
| `total_supply` | - | u128 | Total token supply |
| `balance_of` | address | u128 | Token balance of an address |
| `transfer` | to, amount | Result | Transfer tokens to recipient |
| `approve` | spender, amount | Result | Approve spender allowance |
| `transfer_from` | from, to, amount | Result | Transfer using allowance |
| `allowance` | owner, spender | u128 | Check approved allowance |
| `mint` | to, amount | Result | Mint new tokens (owner only) |
| `burn` | from, amount | Result | Burn tokens |

## Storage Layout

| Slot Range | Purpose |
|------------|---------|
| 0-99 | BaseContract (reserved) |
| 100 | Total supply |
| 101+ | Balances mapping: `keccak256(address \|\| 101)` |
| 200+ | Allowances mapping: `keccak256(spender \|\| keccak256(owner \|\| 200))` |
| 300+ | Token name (string storage) |
| 400+ | Token symbol (string storage) |

All values are stored as 32-byte arrays with u128 values in little-endian format.

## Deploying a SAVITRI-20 Token

### Via Contract Runtime (Rust)

```rust
use savitri_contracts::contracts::standards::savitri20::SAVITRI20;

// Initialize a new token
SAVITRI20::initialize(
    &mut contract_storage,
    &storage,
    &owner_address,        // [u8; 32]
    "My Token",            // name
    "MTK",                 // symbol
    1_000_000_000_000_000_000_000_000,  // 1M tokens (18 decimals)
    Some(&mut gas_meter),
)?;
```

### Via SDK

```rust
use savitri_sdk::{TransactionBuilder, Wallet};

let wallet = Wallet::from_private_key_hex("your_key")?;

// Deploy token contract
let deploy_tx = TransactionBuilder::new()
    .data(savitri20_bytecode)
    .value(0)
    .nonce(nonce)
    .fee(5_000_000_000_000_000)
    .build_and_sign(&wallet)?;
```

## Token Operations

### Transfer

```rust
// Direct transfer
SAVITRI20::transfer(
    &mut contract_storage,
    &storage,
    &sender,        // [u8; 32]
    &recipient,     // [u8; 32]
    1000_000_000_000_000_000,  // 1000 tokens
    &mut event_system,
    Some(&mut gas_meter),
)?;
```

### Approve and TransferFrom

```rust
// Approve spender
SAVITRI20::approve(
    &mut contract_storage,
    &storage,
    &owner,
    &spender,
    5000_000_000_000_000_000,  // 5000 token allowance
    &mut event_system,
    Some(&mut gas_meter),
)?;

// Transfer using allowance
SAVITRI20::transfer_from(
    &mut contract_storage,
    &storage,
    &spender,       // caller (must have allowance)
    &owner,         // from
    &recipient,     // to
    1000_000_000_000_000_000,
    &mut event_system,
    Some(&mut gas_meter),
)?;
```

### Query Balances

```rust
let balance = SAVITRI20::balance_of(&contract_storage, &storage, &address)?;
let supply = SAVITRI20::total_supply(&contract_storage, &storage)?;
let allowed = SAVITRI20::allowance(&contract_storage, &storage, &owner, &spender)?;
```

## Events

| Event | Fields | Description |
|-------|--------|-------------|
| `Transfer` | from, to, amount | Token transfer |
| `Approval` | owner, spender, amount | Allowance approval |
| `Mint` | to, amount | New tokens minted |
| `Burn` | from, amount | Tokens burned |

## Security

- **Non-zero address check**: Transfers to/from the zero address are rejected
- **Balance validation**: Transfers fail if sender has insufficient balance
- **Allowance check**: `transfer_from` verifies and decrements allowance atomically
- **Overflow protection**: All arithmetic uses checked operations
- **Keccak256 slot hashing**: Prevents storage slot collisions between different mappings
