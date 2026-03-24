# Emission Schedule

> 50-year token emission model for $SAVI.

## Overview

Total supply: **2,000,000,000 SAVI** (fixed, no perpetual inflation).
Staking pool: **840,000,000 SAVI** (42%) — maximum allocation ceiling.
All emissions are hardcoded at protocol level — no human controls the release.

## Staking Emission

The staking pool is a **budget ceiling**, not a distribution target. Actual emission is determined by network activity and the halving schedule.

### Halving Phases

| Phase | Years | Halving Interval | Mechanism |
|---|---|---|---|
| **Bootstrap** | 0 – 10 | Every 2 years | 5 halvings, rapid decay |
| **Maturity** | 10 – 30 | Every 5 years | 4 halvings, stabilization |
| **Long-tail** | 30 – 50 | Every 10 years | 2 halvings, minimal emission |

### TVL-Triggered Acceleration

If network TVL exceeds $100M, the next halving fires early. This self-regulates inflation based on actual network growth.

### Staking Emission Table

| Period | Annual Rate | Period Emission | Cumulative |
|---|---|---|---|
| Y0–Y1 | 21M (ramp-up) | 21M | 21M |
| Y1–Y2 | 42M | 42M | 63M |
| Y2–Y4 | 21M | 42M | 105M |
| Y4–Y6 | 10.5M | 21M | 126M |
| Y6–Y8 | 5.25M | 10.5M | 136.5M |
| Y8–Y10 | 2.625M | 5.25M | 141.75M |
| Y10–Y15 | 1.3125M | 6.56M | 148.3M |
| Y15–Y20 | 0.656M | 3.28M | 151.6M |
| Y20–Y30 | 0.328–0.164M | 2.46M | 154.1M |
| Y30–Y50 | 0.082–0.041M | 1.23M | 155.3M |

Total distributed over 50 years: **~155M SAVI** (18.5% of the 840M pool).

### Undistributed Pool Burn

Starting Year 5, 5% of the remaining undistributed staking pool is burned annually:

| Year | Pool Remaining | 5% Burn | Cumulative Burned |
|---|---|---|---|
| Y5 | 714M | 35.7M | 35.7M |
| Y10 | 611M | 30.5M | 166M |
| Y20 | 351M | 17.6M | 367M |
| Y30 | 201M | 10.1M | 486M |
| Y50 | 66M | 3.3M | 580M |

## Non-Staking Vesting

| Category | Total | TGE | Remaining | Cliff | Linear |
|---|---|---|---|---|---|
| Public Sale | 60M | 60M (100%) | 0 | — | — |
| Liquidity | 140M | 70M (50%) | 70M | — | 60 months |
| Marketing | 120M | 30M (25%) | 90M | 3 months | 72 months |
| Community | 60M | 6M (10%) | 54M | 3 months | 36 months |
| Pre-Sale | 120M | 18M (15%) | 102M | 4 months | 18 months |
| Private | 120M | 12M (10%) | 108M | 9 months | 24 months |
| Seed | 60M | 6M (10%) | 54M | 12 months | 24 months |
| Ecosystem | 180M | 18M (10%) | 162M | 12 months | 72 months |
| Team | 220M | 0 (0%) | 220M | 12 months | 60 months |
| DAO | 80M | 0 (0%) | 80M | 12 months | 72 months |

Non-staking vesting completes by Year 7 (month 84).

## Circulating Supply Projection

| Year | Non-Staking Cumulative | Staking Cumulative | Total Circulating | % of 2B |
|---|---|---|---|---|
| TGE | 220M | 0 | 220M | 11.0% |
| Year 1 | 323M | 21M | 344M | 17.2% |
| Year 2 | 597M | 63M | 660M | 33.0% |
| Year 3 | 769M | 84M | 853M | 42.7% |
| Year 5 | 1,007M | 116M | 1,123M | 56.2% |
| Year 7 | 1,155M | 133M | 1,288M | 64.4% |
| Year 10 | 1,160M | 142M | 1,302M | 65.1% |
| Year 20 | 1,160M | 152M | 1,312M | 65.6% |
| Year 50 | 1,160M | 155M | 1,315M | 65.8% |

## Effective Maximum Supply

After accounting for staking pool burns, the effective maximum supply decreases over time:

| Year | Effective Max Supply | % of Original 2B |
|---|---|---|
| TGE | 2,000M | 100% |
| Year 5 | 1,972M | 98.6% |
| Year 10 | 1,844M | 92.2% |
| Year 20 | 1,675M | 83.8% |
| Year 50 | ~1,420M | 71.0% |

By Year 50, approximately 29% of the original supply is permanently removed.
