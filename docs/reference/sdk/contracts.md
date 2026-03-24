# Contract Client

The `ContractClient` provides high-level helpers for interacting with smart contracts, oracles, and governance on the Savitri Network.

## Setup

```rust
use savitri_sdk::{ContractClient, Wallet, RpcClient};

let wallet = Wallet::from_private_key_hex("your_private_key")?;
let rpc = RpcClient::from_url("http://localhost:8545")?;

let contract = ContractClient::new(rpc, wallet);

// Or from URL directly
let contract = ContractClient::from_url_and_wallet(
    "http://localhost:8545",
    wallet,
)?;
```

## Contract Calls

### Generic Contract Call

```rust
let tx_hash = contract.call_contract(
    &"contract_address_64hex".to_string(),  // contract address
    b"function_name",                        // function selector
    b"encoded_arguments",                    // ABI-encoded args
    Some(0),                                 // value to send (0 for read-like)
).await?;

println!("TX hash: {}", tx_hash);
```

The `call_contract` method:
1. Builds the call data (`function_selector || args`)
2. Creates a transaction via `TransactionBuilder`
3. Signs it with the wallet
4. Submits via `savitri_sendRawTransaction`
5. Returns the transaction hash

## Oracle Client

Access the oracle system for external data feeds:

```rust
let oracle = contract.oracle();

// Request data from an oracle
let tx_hash = oracle.request_data(
    &"oracle_address".to_string(),
    "temperature",          // data type
    b"sensor_id_001",       // parameters
).await?;

// Submit a response (as oracle provider)
let tx_hash = oracle.submit_response(
    &"oracle_address".to_string(),
    request_id,             // u64
    b"25.5",                // response data
).await?;

// Verify data
let is_valid = oracle.verify_data(
    &"oracle_address".to_string(),
    b"data_to_verify",
).await?;
```

## Governance Client

Interact with the on-chain governance system:

```rust
let gov = contract.governance();

// Create a proposal
let tx_hash = gov.create_proposal(
    &"governance_address".to_string(),
    "Upgrade consensus parameters",    // title
    "Increase block size to 2MB",      // description
    604800,                            // voting period (seconds)
).await?;

// Vote on a proposal
let tx_hash = gov.vote(
    &"governance_address".to_string(),
    proposal_id,    // u64
    true,           // true = support, false = oppose
).await?;

// Execute an approved proposal
let tx_hash = gov.execute(
    &"governance_address".to_string(),
    proposal_id,
).await?;

// Check proposal status
let status = gov.get_proposal_status(
    &"governance_address".to_string(),
    proposal_id,
).await?;

println!("Proposal #{}: {}", status.id, status.status);
println!("Votes: {} for, {} against", status.votes_for, status.votes_against);
println!("Executed: {}", status.executed);
```

### ProposalStatus Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | u64 | Proposal identifier |
| `title` | String | Proposal title |
| `votes_for` | u64 | Number of votes in favor |
| `votes_against` | u64 | Number of votes against |
| `status` | String | `"active"`, `"passed"`, `"rejected"`, `"executed"` |
| `executed` | bool | Whether the proposal has been executed |
