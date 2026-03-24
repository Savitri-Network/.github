# Tokenomics

## Token System

Savitri Network uses a unified token (SAVT) for all operations on testnet.

### SAVT (Savitri Test Token)

| Property | Value |
|----------|-------|
| Symbol | SAVT / TEST |
| Decimals | 18 |
| Total Supply | 100,000,000 SAVT (100M) |
| Treasury | Address `0x0...01` |

## Fee Structure

SAVT is used for ALL on-chain operations: transfers, contract deployment, governance, NFT operations, and IoT data.

### Transaction Fees

| Operation | Fee (SAVT) | Raw Value (18 decimals) |
|-----------|-----------|------------------------|
| Normal transfer | 0.001 | `1_000_000_000_000_000` |
| Contract deploy/call | 0.005 | `5_000_000_000_000_000` |
| IoT data submission | 0.00005 | `50_000_000_000_000` |

### Fee Distribution

| Recipient | Share | Purpose |
|-----------|-------|---------|
| Burn | 50% | Deflationary pressure |
| Treasury | 30% | Network development fund |
| Block proposer | 20% | Proposer incentive |

### Congestion Pricing

Fees increase dynamically based on network load:

| Trigger | Threshold |
|---------|-----------|
| Mempool size | > 1,000 transactions |
| Block time | > 5,000 ms |
| Throughput | > 100 TPS |
| Gas utilization | > 80% |

Maximum fee multiplier: 100x base fee.

### Fee Sustainability

At 5,000 TPS sustained:
- Daily burn: ~216,000 SAVT (50% of fees)
- Supply exhaustion estimate: ~462 days
- Minimum balance for transactions: 0.001 SAVT

## Rewards

### Node Rewards per Epoch

| Node Type | Base Reward | Condition |
|-----------|-------------|-----------|
| Lightnode | 50 SAVT | Min 80% uptime |
| Masternode | 100 SAVT | Min 80% uptime |

### Block Bonuses

| Bonus | Amount |
|-------|--------|
| Block proposer | 20 SAVT per block |
| Block validator | 5 SAVT per block validated |

### PoU Tier Bonuses

| Tier | Min PoU Score | Reward Multiplier | Vote Tokens |
|------|--------------|-------------------|-------------|
| Bronze | 300 | 1.0x | 10 |
| Silver | 500 | 1.5x | 25 |
| Gold | 700 | 2.0x | 50 |
| Platinum | 900 | 3.0x | 100 |

Nodes below score 300 do not receive rewards.

### Reward Parameters

| Parameter | Value |
|-----------|-------|
| Distribution period | 24 hours |
| Minimum uptime | 80% |
| Minimum PoU score | 300 |
| Maximum multiplier | 3.0x |
| Decay rate (inactive) | 0.95 per epoch |
| Minimum activity | 10 blocks per epoch |

## Faucet (Testnet)

| Parameter | Value |
|-----------|-------|
| Amount per claim | 5 SAVT |
| Cooldown | 24 hours |
| Daily limit per address | 5,000 SAVT |
| Total faucet supply | 1,000,000 SAVT |
| Captcha | Not required |

## Mainnet Conversion

Testnet rewards convert to mainnet tokens at launch:

| Parameter | Value |
|-----------|-------|
| Base rate | 1 REWARD = 0.8 MAINNET |
| High PoU bonus (avg >= 900) | +20% |
| Medium PoU bonus (avg >= 750) | +10% |
| Minimum epochs | 10 epochs participated |

Conversion is currently disabled (`enabled = false` in testnet config).
