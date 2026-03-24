# Smart Contracts

Savitri Network provides a native smart contract runtime with built-in token standards, governance, and oracle support.

## Contract Runtime

Contracts run in a sandboxed environment with:

- **Gas metering**: Every operation consumes gas to prevent infinite loops
- **Storage slots**: Key-value storage using Keccak256-hashed slot addresses
- **Event system**: Contracts emit events for external consumers
- **Memory monitoring**: Runtime memory limits prevent DoS
- **Upgrade support**: Contracts can be upgraded via governance

## Token Standards

| Standard | Type | Equivalent | Description |
|----------|------|------------|-------------|
| [SAVITRI-20](savitri-20.md) | Fungible | ERC-20 | Token transfers, approvals, allowances |
| [SAVITRI-721](savitri-721.md) | Non-Fungible (NFT) | ERC-721 | Unique token ownership, transfers, metadata |
| [SAVITRI-1155](savitri-1155.md) | Multi-Asset | ERC-1155 | Batch operations, mixed fungible/non-fungible |

## Built-in Contracts

### Governance

On-chain proposal and voting system:
- Create proposals with title, description, and voting period
- Vote with support/oppose
- Execute approved proposals
- FL (Federated Learning) proposal specialization

See [Governance](../governance/overview.md) for details.

### Oracle

External data feed system:
- Register oracle providers
- Request and submit data
- Proof verification
- Schema validation

See [Oracle System](oracle.md) for details.

## Contract Interaction via SDK

### Using ContractClient

```rust
use savitri_sdk::{ContractClient, Wallet, RpcClient};

let wallet = Wallet::from_private_key_hex("your_key")?;
let contract = ContractClient::from_url_and_wallet("http://localhost:8545", wallet)?;

// Call a contract function
let tx_hash = contract.call_contract(
    &"contract_address".to_string(),
    b"transfer",
    &encoded_args,
    Some(0),
).await?;
```

### Using TransactionBuilder

```rust
use savitri_sdk::TransactionBuilder;

let tx = TransactionBuilder::new()
    .to("contract_address")
    .data(call_data)     // function selector + encoded args
    .value(0)
    .nonce(nonce)
    .fee(5_000_000_000_000_000)  // 0.005 SAVT contract fee
    .build_and_sign(&wallet)?;
```

## Storage Model

Contracts use a slot-based storage model similar to Ethereum:

- **Slots 0-99**: Reserved for `BaseContract` (owner, metadata)
- **Slot 100+**: Contract-specific storage

Slot addresses are derived using Keccak256 hashing to prevent collisions:

```
balance_slot = keccak256(address || SLOT_BALANCES_BASE)
allowance_slot = keccak256(spender || keccak256(owner || SLOT_ALLOWANCES_BASE))
```

All values are stored as 32-byte arrays (little-endian for numeric types).

## Gas Costs

| Operation | Gas Cost | Fee (SAVT) |
|-----------|---------|------------|
| Contract deploy | Variable | 0.005 base |
| Contract call | Variable | 0.005 base |
| Storage write (SSTORE) | 20,000 | Included in base |
| Storage read (SLOAD) | 200 | Included in base |
| Token transfer | ~50,000 | 0.005 |
| NFT mint | ~80,000 | 0.005 |
| Governance vote | ~30,000 | 0.005 |

## Contract Deployment

Contracts are deployed via special transactions with `to = None` and the contract bytecode in the `data` field:

```rust
let deploy_tx = TransactionBuilder::new()
    // No .to() -- indicates contract deployment
    .data(contract_bytecode)
    .value(0)
    .nonce(nonce)
    .fee(5_000_000_000_000_000)
    .build_and_sign(&wallet)?;
```
