# SAVITRI-1155: Multi-Asset Token Standard

SAVITRI-1155 (SMA -- Savitri Multi Asset) is the multi-asset token standard for the Savitri Network, equivalent to Ethereum's ERC-1155. It supports both fungible and non-fungible tokens in a single contract with efficient batch operations.

## Interface

| Function | Params | Returns | Description |
|----------|--------|---------|-------------|
| `balance_of` | owner, id | u128 | Balance of a specific token for an owner |
| `balance_of_batch` | owners[], ids[] | u128[] | Batch balance query |
| `safe_transfer_from` | from, to, id, amount, data | Result | Transfer tokens |
| `safe_batch_transfer_from` | from, to, ids[], amounts[], data | Result | Batch transfer |
| `set_approval_for_all` | operator, approved | Result | Approve operator for all tokens |
| `is_approved_for_all` | owner, operator | bool | Check operator approval |

## Storage Layout

### Balance Storage (Slot 100+)

Nested mapping: `balances[owner][id]`

```
hash1 = keccak256(owner || 100)
hash2 = keccak256(id || hash1)
slot  = first 8 bytes of hash2 as u64
```

### Operator Approval Storage (Slot 200+)

Nested mapping: `operator_approvals[owner][operator]`

```
hash1 = keccak256(owner || 200)
hash2 = keccak256(operator || hash1)
slot  = first 8 bytes of hash2 as u64
```

This layout provides:
- **Uniform distribution**: Keccak256 ensures uniform slot distribution
- **No collisions**: Negligible collision probability
- **Efficient batch queries**: Each slot is calculated independently
- **Cache-friendly**: ContractStorage caches reads

## Usage

### Single Transfer

```rust
use savitri_contracts::contracts::standards::savitri1155::SAVITRI1155;

SAVITRI1155::safe_transfer_from(
    &mut contract_storage,
    &storage,
    &from,          // [u8; 32]
    &to,            // [u8; 32]
    1,              // token id
    100,            // amount
    &[],            // data
    &mut event_system,
    Some(&mut gas_meter),
)?;
```

### Batch Transfer

```rust
SAVITRI1155::safe_batch_transfer_from(
    &mut contract_storage,
    &storage,
    &from,
    &to,
    &[1, 2, 3],           // token ids
    &[100, 50, 1],        // amounts
    &[],                   // data
    &mut event_system,
    Some(&mut gas_meter),
)?;
```

### Batch Balance Query

```rust
let balances = SAVITRI1155::balance_of_batch(
    &contract_storage,
    &storage,
    &[owner1, owner2, owner3],
    &[token_id_1, token_id_2, token_id_3],
)?;
```

### Operator Approval

```rust
// Approve an operator for all tokens
SAVITRI1155::set_approval_for_all(
    &mut contract_storage,
    &storage,
    &owner,
    &operator,
    true,           // approved
    &mut event_system,
    Some(&mut gas_meter),
)?;

// Check approval
let approved = SAVITRI1155::is_approved_for_all(
    &contract_storage,
    &storage,
    &owner,
    &operator,
)?;
```

## Events

| Event | Fields | Description |
|-------|--------|-------------|
| `TransferSingle` | operator, from, to, id, amount | Single token transfer |
| `TransferBatch` | operator, from, to, ids[], amounts[] | Batch transfer |
| `ApprovalForAll` | owner, operator, approved | Operator approval changed |

## Use Cases

| Scenario | Token Type | Example |
|----------|-----------|---------|
| In-game currency | Fungible (id=1) | 1000 gold coins |
| Unique items | Non-fungible (amount=1) | Legendary sword #42 |
| Semi-fungible | Limited supply | 50 copies of a rare card |
| Rewards | Fungible | PoU reward tokens |
| Certificates | Non-fungible | Validator certification |

## Batch Optimization

The SAVITRI-1155 implementation optimizes batch operations by:

1. **Pre-calculating all slots** before reading storage
2. **Leveraging ContractStorage cache** to avoid repeated DB reads
3. **Independent slot calculation** enabling future parallel reads
4. **Minimizing address decoding** by reusing decoded addresses across operations
