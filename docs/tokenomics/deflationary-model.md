# Deflationary Model

> How Savitri achieves net deflation through multiple burn mechanisms.

## Overview

Savitri transitions from inflationary (vesting-driven, Y0–Y5) to **net deflationary between Year 6 and Year 7** under moderate network adoption. Four independent burn mechanisms work in concert.

## Burn Mechanisms

### 1. Transaction Fee Burn (50% of all fees)

Every transaction burns 50% of its fee permanently. This is the primary deflation driver.

| Network Activity | Annual Burn |
|---|---|
| 10K TX/day (Y1) | ~255,000 SAVI |
| 1M TX/day (Y5) | ~25,500,000 SAVI |
| 20M TX/day (Y10) | ~182,500,000 SAVI |

### 2. AI/IoT Micro-Burn (50% of service fees)

AI inference queries and IoT data certification fees have an additional 50% "utility lock" burn. As AI/IoT adoption grows, this accelerates scarcity.

### 3. Staking Pool Burn (5%/year from Y5)

Starting Year 5, 5% of the undistributed staking pool is burned annually. This reduces the theoretical maximum supply regardless of network activity.

Over 45 years, approximately **580M SAVI** are burned from the staking pool — tokens that would never have entered circulation are permanently destroyed.

### 4. Volume-Based Adaptive Burn (0.1–1%)

An additional burn scales with network transaction volume:

- Minimum: 0.1% (low activity periods)
- Maximum: 1.0% (high activity periods)
- Formula: `burn_rate = 0.1% + (0.9% × daily_volume / 1B)`

## Net Inflation/Deflation Timeline

| Year | New Tokens (vesting + staking) | Estimated Burn | Net Change | Status |
|---|---|---|---|---|
| Y1 | ~124M | ~0.3M | +123.7M | Inflationary |
| Y2 | ~316M | ~2.5M | +313.5M | Inflationary (peak) |
| Y3 | ~214M | ~12.8M | +201M | Inflationary (declining) |
| Y5 | ~134M | ~72M | +62M | Low inflation |
| Y7 | ~57M | ~120M | **-63M** | **Deflationary** |
| Y10 | ~5M | ~183M | **-178M** | **Strongly deflationary** |

## Effective Supply Over Time

| Year | Circulating Supply | Max Supply (after burns) | Effective Scarcity |
|---|---|---|---|
| TGE | 220M | 2,000M | 11% circulating |
| Y5 | 1,123M | 1,972M | 57% circulating |
| Y10 | 1,302M | 1,844M | 71% of effective supply |
| Y20 | 1,312M | 1,675M | 78% of effective supply |
| Y50 | ~1,315M | ~1,420M | 93% of effective supply |

## Design Philosophy

The deflationary model is **deterministic and protocol-enforced**. All burn parameters are hardcoded — no governance vote or team decision can change the burn rate without a network-wide hard fork. This provides mathematical guarantees to token holders: if the network is used, the supply shrinks. No trust required.
