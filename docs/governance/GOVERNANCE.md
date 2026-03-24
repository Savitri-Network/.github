# Governance

Savitri Network features on-chain governance enabling token holders to propose, vote on, and execute network changes.

## Overview

The governance system supports:
- **Proposal creation** with title, description, and voting period
- **Token-weighted voting** (support/oppose)
- **Automatic execution** of approved proposals
- **Federated Learning (FL) proposals** for AI model governance
- **Vote token rewards** based on PoU tier

## Proposal Lifecycle

```
1. Create Proposal
       │
2. Voting Period (configurable, e.g., 7 days)
       │
3. Tally Votes
       │
   ├── Approved (majority support) → 4. Execute
   └── Rejected → Archived
```

## Creating a Proposal

### Via SDK

```rust
use savitri_sdk::{ContractClient, Wallet};

let contract = ContractClient::from_url_and_wallet(url, wallet)?;
let gov = contract.governance();

let tx = gov.create_proposal(
    &governance_address,
    "Increase block size",
    "Proposal to increase max block transactions from 32 to 64",
    604800,  // 7-day voting period (seconds)
).await?;
```

### Via TransactionBuilder

```rust
use savitri_sdk::{TransactionBuilder, GovernanceAction};

let tx = TransactionBuilder::new()
    .create_fl_proposal(
        "governance_contract_address",
        "Deploy model v2",
        "Replace current FL model with GPT-4 fine-tuned variant",
        604800,
    )
    .nonce(nonce)
    .fee(5_000_000_000_000_000)
    .build_and_sign(&wallet)?;
```

## Voting

```rust
// Vote in favor
let tx = gov.vote(&governance_address, proposal_id, true).await?;

// Vote against
let tx = gov.vote(&governance_address, proposal_id, false).await?;

// Via TransactionBuilder
let tx = TransactionBuilder::new()
    .governance_call(
        "governance_address",
        proposal_id,
        GovernanceAction::Vote(true),
    )
    .nonce(nonce)
    .build_and_sign(&wallet)?;
```

## Executing Proposals

Approved proposals can be executed by any participant:

```rust
let tx = gov.execute(&governance_address, proposal_id).await?;
```

## Checking Status

```rust
let status = gov.get_proposal_status(&governance_address, proposal_id).await?;

println!("Title: {}", status.title);
println!("Status: {}", status.status);  // "active", "passed", "rejected", "executed"
println!("Votes: {} for / {} against", status.votes_for, status.votes_against);
println!("Executed: {}", status.executed);
```

## Vote Tokens

Validators earn vote tokens based on their PoU tier:

| Tier | PoU Score | Vote Tokens per Epoch |
|------|-----------|----------------------|
| Bronze | 300-499 | 10 |
| Silver | 500-699 | 25 |
| Gold | 700-899 | 50 |
| Platinum | 900-1000 | 100 |

Vote tokens determine voting weight in governance proposals.

## Governance Contracts

The governance system consists of several on-chain modules:

| Module | File | Purpose |
|--------|------|---------|
| Proposals | `governance/proposals.rs` | Proposal creation and management |
| Voting | `governance/voting.rs` | Vote casting and tallying |
| Execution | `governance/execution.rs` | Approved proposal execution |
| Deposit | `governance/deposit.rs` | Proposal deposits (anti-spam) |
| Vote Token | `governance/vote_token.rs` | Vote token issuance and tracking |
| FL Proposals | `governance/fl_proposals.rs` | Federated Learning-specific proposals |

## FL Governance

Federated Learning proposals are specialized governance actions for managing distributed AI training:

- **Model registry**: Propose new models for training
- **Job lifecycle**: Create, manage, and finalize FL jobs
- **Model updates**: Vote on model version upgrades
- **Resource allocation**: Allocate compute resources via governance
