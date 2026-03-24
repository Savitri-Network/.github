# Oracle System

The Savitri Oracle system provides verified external data feeds to on-chain contracts. It supports data requests, provider responses, proof verification, and schema validation.

## Architecture

```
External Data Source
       │
Oracle Provider (off-chain)
       │
   submit_response()
       │
Oracle Registry Contract (on-chain)
       │
   verify_data() + proof
       │
Consumer Contract
```

## Components

### Oracle Registry (`oracle_registry.rs`)

Central registry for oracle providers and data feeds:
- Provider registration and management
- Data feed configuration
- Access control

### Oracle Feed (`oracle/feed.rs`)

Data feed management:
- Feed creation and updates
- Historical data tracking
- Aggregation support

### Oracle Proof (`oracle/proof.rs`)

Cryptographic verification of oracle data:
- Signature-based proofs
- Multi-provider consensus
- Tamper detection

### Oracle Schema (`oracle/schema.rs`)

Data format validation:
- Schema definitions for data types
- Input/output validation
- Type checking

## Using Oracles via SDK

### Request Data

```rust
use savitri_sdk::ContractClient;

let contract = ContractClient::from_url_and_wallet(url, wallet)?;
let oracle = contract.oracle();

// Request temperature data
let tx = oracle.request_data(
    &oracle_address,
    "temperature",
    b"sensor_001",
).await?;
```

### Submit Response (Provider)

```rust
let tx = oracle.submit_response(
    &oracle_address,
    request_id,         // u64
    b"25.5",            // response bytes
).await?;
```

### Verify Data

```rust
let is_valid = oracle.verify_data(
    &oracle_address,
    b"data_to_verify",
).await?;
```

### Using TransactionBuilder

```rust
use savitri_sdk::TransactionBuilder;

let tx = TransactionBuilder::new()
    .oracle_call(
        "oracle_contract_address",
        "request_data",
        b"params",
    )
    .nonce(nonce)
    .fee(5_000_000_000_000_000)
    .build_and_sign(&wallet)?;
```

## Oracle Data Types

| Type | Format | Use Case |
|------|--------|----------|
| Price feed | u128 (18 decimals) | Token prices, exchange rates |
| Sensor data | bytes | IoT temperature, humidity, etc. |
| Random number | [u8; 32] | Verifiable randomness |
| Timestamp | u64 | External event timestamps |
| Binary | bool | Yes/no outcomes |

## Integration with IoT

The oracle system integrates with the IoT connector for sensor data:

```rust
let tx = TransactionBuilder::new()
    .oracle_call(
        "iot_oracle_address",
        "submit_sensor_data",
        &encode_sensor_data(sensor_id, value, timestamp),
    )
    .nonce(nonce)
    .fee(50_000_000_000_000)  // IoT rate: 0.00005 SAVT
    .build_and_sign(&wallet)?;
```
