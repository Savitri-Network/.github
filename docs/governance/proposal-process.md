# Proposal Process

> How to submit, discuss, and vote on governance proposals in Savitri Network.

## Proposal Lifecycle

```
Draft → Discussion → Formal Submission → Voting Period → Execution / Rejection
```

### 1. Draft

Any $VOTE token holder can draft a proposal. Proposals cover:
- Protocol parameter changes (fees, block size, consensus weights)
- Treasury spending (grants, partnerships, buy-backs)
- Network upgrades (hard forks, feature activation)
- Slashing parameter adjustments

### 2. Discussion

- Post the draft on the governance forum
- Minimum discussion period: 7 days
- Community feedback and iteration
- Proposal author can modify during discussion

### 3. Formal Submission

Requirements to submit:
- Minimum $VOTE stake: governance-defined threshold
- Proposal deposit: refunded if proposal reaches quorum, burned if not
- Clear specification of changes with before/after parameters

### 4. Voting Period

| Parameter | Value |
|---|---|
| Voting duration | 7 days |
| Quorum | 20% of total $VOTE supply |
| Approval threshold | >50% of votes cast |
| Vote weight | Proportional to $VOTE balance |
| Vote options | Yes / No / Abstain |

### 5. Execution

- Approved proposals are executed automatically by the protocol
- Parameter changes take effect at next epoch boundary
- Treasury transfers require 3/5 multisig confirmation + DAO vote
- Network upgrades follow a signaling + activation schedule

## Proposal Types

| Type | Quorum | Threshold | Timelock |
|---|---|---|---|
| Parameter change | 20% | >50% | 1 epoch |
| Treasury spend (<50K SAVI) | 20% | >50% | 24 hours |
| Treasury spend (>50K SAVI) | 30% | >60% | 7 days |
| Network upgrade | 40% | >66.7% | 14 days |
| Emergency action | 10% | >75% | Immediate |

## $VOTE Token

$VOTE is minted by staking $SAVI. It is non-transferable and used exclusively for governance:
- 1 staked SAVI = 1 VOTE per epoch
- Longer staking duration = bonus VOTE multiplier (+2% per 6 months)
- Unstaking burns the corresponding $VOTE
- Cannot be bought, sold, or traded — only earned through network participation
