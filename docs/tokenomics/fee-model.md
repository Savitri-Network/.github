# Fee Model

> Transaction fee structure, distribution, and dynamic pricing.

## Fee Schedule

| Transaction Type | Base Fee | Use Case |
|---|---|---|
| Standard transfer | $0.0035 | Wallet-to-wallet payments |
| Smart contract call | $0.0175 | dApp interactions, DAO proposals |
| IoT data packet | $0.000125 | High-frequency sensor data |
| AI inference query | $0.0025 | Decentralized ML services |

### Dynamic Fee Multiplier

Fees adjust dynamically during network congestion, up to 100x the base fee.

Congestion thresholds:

| Metric | Threshold | Effect |
|---|---|---|
| Mempool size | > 1,000 pending TX | Fee multiplier increases |
| Block time deviation | > 5,000ms | Fee multiplier increases |
| Throughput | < 100 TPS | Fee multiplier increases |
| Gas utilization | > 80% | Fee multiplier increases |

## Fee Distribution

Every transaction fee is split into four streams:

```
  50%  burned permanently      →  reduces circulating supply
  10%  network treasury        →  perpetual ecosystem funding
  20%  block proposer          →  direct validator incentive
  20%  staking reward pool     →  distributed to active validators
```

### Treasury Revenue Projections

The 10% treasury fee creates a self-sustaining funding mechanism:

| Year | Est. Daily TX | Annual Treasury Revenue |
|---|---|---|
| Y1 | 10,000 | ~51,000 SAVI |
| Y3 | 200,000 | ~876,000 SAVI |
| Y5 | 1,000,000 | ~3,650,000 SAVI |
| Y10 | 20,000,000 | ~43,800,000 SAVI |

At scale (20M+ daily transactions), the treasury generates more annual funding than any pre-allocated ecosystem grant pool.

## IoT Micro-Fee Design

IoT transactions use a dedicated transaction type with fees 28x lower than standard transfers. This enables:

- Continuous sensor data streams at minimal cost
- Batch certification of IoT device data
- Mesh network revenue tracking

AI/IoT service fees include an additional 50% micro-burn applied directly to the service fee, accelerating deflation as AI/IoT adoption grows.
