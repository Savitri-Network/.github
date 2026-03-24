# Contract Runtime

The Savitri contract runtime executes smart contracts in a sandboxed environment with gas metering, storage isolation, and event emission.

## Execution Flow

```
Contract Call TX arrives
    │
    ▼
BaseContract validation
    │ check paused, owner, reserved slots
    ▼
Gas Meter initialization
    │ gas_limit from TX fee
    ▼
ContractStorage setup
    │ load contract state, init cache
    ▼
Function dispatch
    │ match function_selector → handler
    ▼
Execution (with gas tracking)
    │ SLOAD, SSTORE, events, calls
    ▼
Result
    ├── Success → commit storage, emit events
    └── Failure → revert all changes
```

## BaseContract

All contracts extend BaseContract. Slots 0-99 are reserved:

| Slot | Field | Size | Description |
|------|-------|------|-------------|
| 0 | `owner` | 32 bytes | Contract owner address |
| 1 | `version` | u64 | Contract version |
| 2 | `governance_hook` | bool | Governance integration enabled |
| 3 | `fee_hook` | bool | Custom fee logic enabled |
| 4 | `paused` | bool | Contract paused state |
| 5-99 | Reserved | - | Future use |

### BaseContract Functions

| Function | Access | Description |
|----------|--------|-------------|
| `owner()` | Public | Get contract owner |
| `transfer_ownership(new_owner)` | Owner only | Transfer ownership |
| `version()` | Public | Get contract version |
| `upgrade(new_version)` | Owner only | Upgrade contract |
| `pause()` | Owner only | Pause all operations |
| `unpause()` | Owner only | Resume operations |

## Gas Metering

### Gas Costs

| Operation | Gas | Description |
|-----------|-----|-------------|
| `SLOAD` | 100 | Storage read |
| `SSTORE` (new) | 20,000 | Storage write (empty slot) |
| `SSTORE` (update) | 5,000 | Storage write (existing slot) |
| `CALL` | 2,300 | Cross-contract call |
| `CREATE` | 32,000 | Contract deployment |
| `TRANSFER` | 300 | Token transfer |
| `LOG` | 375 | Event emission (base) |
| `LOG_TOPIC` | 375 | Per additional topic |
| `LOG_DATA` | 8 | Per byte of event data |
| `CALLDATA` | 16 | Per byte of call data |

### Batch Accounting

The gas meter uses batch accounting (100 operations per batch) to reduce overhead. Gas is committed in batches rather than per-operation.

### Gas Limit

Gas limit is derived from the transaction fee:

```
gas_limit = tx.fee / gas_price
```

If gas runs out during execution, the entire transaction reverts.

## Contract Storage

### Slot-Based Model

Each contract has an independent key-value store where keys are u64 slot numbers and values are 32-byte arrays.

### Slot Derivation

For mappings, slots are derived using Keccak256 to prevent collisions:

```
// Simple value
slot = FIXED_SLOT_NUMBER

// Mapping (address → value)
slot = keccak256(address || base_slot)[0..8] as u64

// Nested mapping (address → address → value)
inner = keccak256(outer_key || base_slot)
slot = keccak256(inner_key || inner)[0..8] as u64
```

### Merkle Tree

Contract storage maintains a Merkle tree for state proof generation.

## Event System

Contracts emit events for external consumers:

```rust
pub struct CustomEvent {
    pub event_type: String,    // e.g., "Transfer", "Approval"
    pub topics: Vec<Vec<u8>>,  // indexed fields
    pub data: Vec<u8>,         // non-indexed data
}
```

Standard events (from BaseContract):
- `OwnershipTransferred(old_owner, new_owner)`
- `ContractUpgraded(old_version, new_version)`
- `Paused(by)`
- `Unpaused(by)`

## Contract Deployment

```
1. Create DeployTransaction (to = None, data = bytecode)
2. Assign contract address (derived from deployer + nonce)
3. Initialize BaseContract (set owner, version)
4. Run contract constructor (initialize function)
5. Store ContractInfo in CF_CONTRACTS
6. Emit ContractDeployed event
```

## Contract Calls

```
1. Parse function_selector from TX data
2. Load contract from CF_CONTRACTS
3. Check BaseContract state (not paused, etc.)
4. Initialize GasMeter with gas_limit
5. Execute function with ContractStorage
6. If success: commit storage changes, deduct gas
7. If failure: revert all changes
```

## EVM Interpreter

The `evm_interpreter` module provides basic EVM bytecode execution for compatibility with Ethereum contracts. This enables deploying Solidity-compiled contracts on Savitri.

## Memory Monitoring

The `memory_monitor` tracks runtime memory usage to prevent DoS:

- Per-contract memory limits
- Allocation tracking
- Automatic termination if limits exceeded

## Parallel Execution

The `parallel` module enables parallel contract execution for independent transactions:

- Dependency analysis between transactions
- Conflict detection on storage slots
- Parallel execution of non-conflicting TXs
- Serial fallback for conflicting TXs

## Contract Tracing

The `tracing` module provides execution traces for debugging:

- Step-by-step execution log
- Storage read/write tracking
- Gas consumption per operation
- Event emission log
