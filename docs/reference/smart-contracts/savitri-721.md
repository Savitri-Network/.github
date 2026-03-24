# SAVITRI-721: NFT Standard

SAVITRI-721 (SNT1 -- Savitri Non-Fungible Token) is the NFT standard for the Savitri Network, equivalent to Ethereum's ERC-721. Each token has a unique ID and a single owner.

## Interface

| Function | Params | Returns | Description |
|----------|--------|---------|-------------|
| `balance_of` | owner | u64 | Number of NFTs owned by address |
| `owner_of` | token_id | address | Owner of a specific token |
| `transfer_from` | from, to, token_id | Result | Transfer an NFT |
| `approve` | to, token_id | Result | Approve address to transfer token |
| `safe_transfer_from` | from, to, token_id | Result | Transfer with safety checks |
| `token_uri` | token_id | String | Get token metadata URI |
| `mint` | to, token_id, uri | Result | Mint a new NFT |

## Storage Layout

| Slot Range | Purpose | Derivation |
|------------|---------|------------|
| 0-99 | BaseContract (reserved) | Direct |
| 100+ | Token owners | `keccak256(100 \|\| token_id)` |
| 200+ | Token balances | `keccak256(200 \|\| address)` |
| 300+ | Token approvals | `keccak256(300 \|\| token_id)` |
| 400+ | Token URIs | `keccak256(400 \|\| token_id)` |

Keccak256 hashing prevents slot collisions between categories (e.g., token_id 100 won't collide with the balances base slot).

## Minting an NFT

```rust
use savitri_contracts::contracts::standards::savitri721::SAVITRI721;

SAVITRI721::mint(
    &mut contract_storage,
    &storage,
    &recipient,             // [u8; 32]
    42,                     // token_id: u64
    "ipfs://Qm.../meta",   // token URI
    &mut event_system,
    Some(&mut gas_meter),
)?;
```

## Transferring

```rust
// Direct transfer
SAVITRI721::transfer_from(
    &mut contract_storage,
    &storage,
    &from,          // current owner
    &to,            // new owner
    42,             // token_id
    &mut event_system,
    Some(&mut gas_meter),
)?;
```

## Querying

```rust
// Who owns token #42?
let owner = SAVITRI721::owner_of(&contract_storage, &storage, 42)?;

// How many NFTs does this address own?
let count = SAVITRI721::balance_of(&contract_storage, &storage, &address)?;

// Get metadata URI
let uri = SAVITRI721::token_uri(&contract_storage, &storage, 42)?;
```

## Events

| Event | Fields | Description |
|-------|--------|-------------|
| `Transfer` | from, to, token_id | NFT transferred |
| `Approval` | owner, approved, token_id | Approval granted |
| `Mint` | to, token_id | New NFT minted |

## Via SDK

```rust
use savitri_sdk::{ContractClient, Wallet};

let contract = ContractClient::from_url_and_wallet("http://localhost:8545", wallet)?;

// Mint NFT via contract call
let tx = contract.call_contract(
    &nft_contract_address,
    b"mint",
    &encode_mint_args(recipient, token_id, uri),
    Some(0),
).await?;
```
