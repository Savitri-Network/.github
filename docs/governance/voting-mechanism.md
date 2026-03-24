# Voting System

The Savitri governance voting system enables token-weighted on-chain decision making.

## Vote Types

| Type | Meaning |
|------|---------|
| `Yes` | Support the proposal |
| `No` | Oppose the proposal |
| `Abstain` | Counted for quorum but not approval |

## Proposal Lifecycle

```
Creation (+ deposit)
    │
Review Period (24 hours)
    │ Pending state, no voting allowed
    ▼
Voting Period (7 days default)
    │ ActiveVoting state
    ▼
Tally
    ├── Quorum met + Approval met → Approved
    ├── Quorum met + Approval not met → Rejected
    └── Quorum not met → Rejected (insufficient participation)
    │
    ▼
Execution (if Approved)
```

## Quorum and Approval

| Threshold | Value | Description |
|-----------|-------|-------------|
| **Quorum** | 10% | Of total vote tokens must participate |
| **Approval** | 65% | Of Yes votes (excluding Abstain) |

### Calculation

```
total_votes = yes_votes + no_votes + abstain_votes
quorum_met = total_votes >= (total_vote_supply * 0.10)
approval = yes_votes / (yes_votes + no_votes) >= 0.65
```

Abstain votes count toward quorum but not toward approval calculation.

## Vote Token Locking

When a user casts a vote, their vote tokens are **locked** for the duration of the voting period. This prevents:
- Double voting (voting with same tokens on multiple proposals)
- Token transfer to vote again from a different address

Tokens are unlocked after the voting period ends.

## Deposit Mechanism

Proposal creation requires a deposit (anti-spam):

- Deposit returned if proposal reaches quorum (regardless of outcome)
- Deposit slashed if proposal fails to reach quorum

## Proposal Actions

| Action | Description |
|--------|-------------|
| `SetParameter` | Change network parameters |
| `TransferTreasury` | Transfer treasury funds |
| `UpgradeContract` | Upgrade a smart contract |
| `SetFlPolicy` | Update FL training parameters |
| `ApproveFlModel` | Approve FL model for production |
| `AbortFlRound` | Emergency abort FL training round |
| `SlashValidator` | Slash a misbehaving validator |
| `Custom` | Custom governance action |

## Voting Results

```rust
pub struct VotingResult {
    pub yes_votes: u64,
    pub no_votes: u64,
    pub abstain_votes: u64,
    pub total_eligible: u64,
    pub quorum_reached: bool,
    pub approved: bool,
    pub participation_rate: f64,
}
```

## Vote Token Distribution

Vote tokens are earned through node participation:

| PoU Tier | Score Range | Vote Tokens / Epoch |
|----------|------------|---------------------|
| Bronze | 300-499 | 10 |
| Silver | 500-699 | 25 |
| Gold | 700-899 | 50 |
| Platinum | 900-1000 | 100 |

## Via SDK

### Cast a Vote

```rust
use savitri_sdk::{ContractClient, Wallet};

let contract = ContractClient::from_url_and_wallet(url, wallet)?;
let gov = contract.governance();

// Vote YES on proposal #42
let tx = gov.vote(&governance_address, 42, true).await?;

// Vote NO
let tx = gov.vote(&governance_address, 42, false).await?;
```

### Check Proposal Status

```rust
let status = gov.get_proposal_status(&governance_address, 42).await?;
println!("Votes: {} for / {} against", status.votes_for, status.votes_against);
println!("Status: {} (executed: {})", status.status, status.executed);
```

### Execute Approved Proposal

Any participant can trigger execution of an approved proposal:

```rust
let tx = gov.execute(&governance_address, 42).await?;
```

## Storage

| Column Family | Key | Value |
|--------------|-----|-------|
| `CF_GOVERNANCE` | proposal_id (u64 LE) | Proposal (bincode) |
| `CF_VOTE_TOKENS` | address (32 bytes) | Vote token balance |

## Timeline

| Phase | Duration | State |
|-------|----------|-------|
| Review | 24 hours | `Pending` |
| Voting | 7 days (configurable) | `ActiveVoting` |
| Execution | Immediate after tally | `Approved` → `Executed` |
