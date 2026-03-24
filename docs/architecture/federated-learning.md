# Federated Learning (FL)

Savitri Network includes an on-chain Federated Learning system for decentralized AI model training. FL contracts manage model registration, training rounds, update submission, reward distribution, and governance.

## Architecture

```
Governance (proposals + voting)
    │
    ▼
Model Registry
    │ register model, assign roles, version tracking
    ▼
Job Lifecycle
    │ create round → open → submit updates → seal → finalize
    ▼
Reward Pool
    │ Merkle-proof reward claims
    ▼
Trainers + Aggregators
```

## Model Registry

Models are registered on-chain with metadata, versioning, and role-based access control.

### Model Registration

```rust
// Register a new model
FlModelRegistry::register_model(
    &mut storage,
    &db,
    &creator,           // [u8; 32] - model creator address
    "sentiment-v1",     // model name
    "ipfs://Qm.../meta", // metadata URI
    "MIT",              // license URI
    "ipfs://Qm.../weights", // initial weights URI
    &mut events,
    Some(&mut gas),
)?;
```

### Model Versioning

Models have immutable version chains. Each new version references its parent:

```
v1 (initial) → v2 (fine-tuned) → v3 (production)
```

Version metadata includes:
- `metadata_uri`: Model architecture description
- `weights_uri`: Trained weights location
- `aggregator`: Address that produced this version
- `parent_version`: Link to previous version

### Role-Based Access Control (RBAC)

| Role | Permissions |
|------|-------------|
| `Creator` | Register models, manage versions, set policies |
| `Viewer` | Read model metadata and versions |
| `Trainer` | Submit training updates to rounds |
| `Aggregator` | Aggregate updates, finalize rounds, produce versions |

Trainer access is controlled via allowlist/denylist per model.

### Policy Management

Each model has configurable policies:

| Policy | Description |
|--------|-------------|
| `access_policy_hash` | Who can access model data |
| `reward_policy_hash` | How rewards are distributed |
| `max_trainers` | Maximum concurrent trainers |
| `aggregator_whitelist` | Approved aggregators |

## Job Lifecycle

FL training is organized into rounds with a defined lifecycle:

```
Planned → Open → Sealed → Finalized
                    ↘ Aborted (via governance)
```

### Round States

| State | Description | Allowed Actions |
|-------|-------------|-----------------|
| `Planned` | Round created, not yet accepting updates | Open |
| `Open` | Accepting trainer updates | Submit update, Seal |
| `Sealed` | No more updates, aggregation in progress | Finalize, Abort |
| `Finalized` | Rewards distributed, round complete | Claim rewards |
| `Aborted` | Round cancelled via governance | None |

### Create a Round

```rust
FlJobLifecycle::create_round(
    &mut storage,
    &db,
    &creator,
    &model_id,          // [u8; 32]
    round_id,           // u64
    reward_pool_amount,  // u128
    &mut events,
    Some(&mut gas),
)?;
```

### Submit Training Update

Trainers submit model updates during the Open phase:

```rust
FlJobLifecycle::submit_update(
    &mut storage,
    &db,
    &trainer,           // [u8; 32]
    &model_id,
    round_id,
    &update_data,       // training update bytes
    nonce,              // replay protection
    &mut events,
    Some(&mut gas),
)?;
```

Replay protection: each trainer has a per-round nonce that must be strictly increasing.

### Seal and Finalize

```rust
// Seal round (stop accepting updates)
FlJobLifecycle::seal_round(&mut storage, &db, &aggregator, &model_id, round_id, ...)?;

// Finalize round (distribute rewards)
FlJobLifecycle::finalize_round(
    &mut storage, &db, &aggregator, &model_id, round_id,
    &merkle_root,       // reward distribution Merkle root
    &new_weights_uri,   // aggregated model weights
    ...
)?;
```

### Claim Rewards

Trainers claim rewards with Merkle proofs:

```rust
FlJobLifecycle::claim_reward(
    &mut storage, &db, &trainer, &model_id, round_id,
    amount,             // reward amount
    &merkle_proof,      // proof of inclusion in reward distribution
    ...
)?;
```

### Treasury Fee

A configurable percentage (`fee_treasury_bps`, max 10000 = 100%) is deducted from the reward pool for the network treasury. Calculation uses fixed-point arithmetic to prevent rounding errors.

## FL Governance Proposals

FL-specific governance actions:

### SetFlPolicy

Propose changes to FL parameters:

```rust
ProposalAction::SetFlPolicy {
    fee_treasury_bps: 500,      // 5% treasury fee
    max_models: 100,            // max registered models
    aggregator_whitelist: vec!["addr1", "addr2"],
}
```

**Validation**: `fee_treasury_bps <= 10000`, `max_models > 0`, non-empty whitelist.

### ApproveFlModel

Approve a model for production use:

```rust
ProposalAction::ApproveFlModel {
    model_id: "abc123...64hex",  // 32-byte hex model ID
}
```

### AbortFlRound

Emergency abort of an active training round:

```rust
ProposalAction::AbortFlRound {
    model_id: "abc123...64hex",
    round_id: 5,
}
```

**Validation**: `round_id > 0`, valid 32-byte model_id.

## Storage Layout

FL contracts use storage slots starting at 100:

| Slot Range | Purpose |
|------------|---------|
| 100+ | Model metadata |
| 200+ | Version chains |
| 300+ | Round state |
| 400+ | Update submissions |
| 500+ | Reward pools |
| 600+ | Role mappings |

## Integration via SDK

```rust
use savitri_sdk::{TransactionBuilder, GovernanceAction};

// Create FL governance proposal
let tx = TransactionBuilder::new()
    .create_fl_proposal(
        "governance_contract_address",
        "Approve sentiment model v2",
        "Production-ready model with 95% accuracy",
        604800,  // 7-day voting period
    )
    .nonce(nonce)
    .fee(5_000_000_000_000_000)
    .build_and_sign(&wallet)?;
```
