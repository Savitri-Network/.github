# IoT Connector

The Savitri IoT Connector enables Internet-of-Things devices to submit sensor data on-chain. It provides device management, data ingestion, edge processing, batch optimization, and enterprise-grade security.

## Architecture

```
IoT Devices (sensors, actuators, gateways)
    │
    ▼
Protocol Handlers (MQTT, CoAP)
    │ parse, validate, authenticate
    ▼
Edge Processing Engine
    │ rules engine, anomaly detection, aggregation
    ▼
Batch Processor
    │ compress, aggregate, schedule
    ▼
Blockchain Interface (savitri_sendRawTransaction)
    │ submit batch as IoT TX (0.00005 SAVT fee)
    ▼
On-chain Oracle Feed
```

## Device Management

### Device Types

| Type | Description | Example |
|------|-------------|---------|
| `Sensor` | Read-only data producer | Temperature sensor |
| `Actuator` | Controllable device | Smart valve |
| `Gateway` | Edge aggregation point | Raspberry Pi hub |
| `EdgeNode` | Local processing unit | Edge ML server |
| `Mobile` | Mobile device | Smartphone |
| `Industrial` | Industrial equipment | PLC/SCADA |

### Device Registration

```rust
pub struct Device {
    pub id: DeviceId,
    pub owner: [u8; 32],          // owner address
    pub public_key: [u8; 32],     // device signing key
    pub device_type: DeviceType,
    pub capabilities: Vec<String>,
    pub status: DeviceStatus,
    pub metadata: HashMap<String, String>,
}
```

Devices are registered on-chain with their public key for authentication. Groups of devices can be managed collectively.

### Device Status

| Status | Description |
|--------|-------------|
| `Active` | Normal operation |
| `Inactive` | Temporarily offline |
| `Suspended` | Suspended by owner/governance |
| `Banned` | Permanently revoked |

## Data Ingestion

### Protocol Support

| Protocol | Handler | Use Case |
|----------|---------|----------|
| **MQTT** | `MQTTHandler` | Low-power sensors, pub/sub |
| **CoAP** | `CoAPHandler` | Constrained devices, UDP |

Both implement the `ProtocolHandler` trait:

```rust
pub trait ProtocolHandler: Send + Sync {
    async fn handle_connection(&self, stream: TcpStream) -> Result<()>;
    fn parse_data(&self, raw: &[u8]) -> Result<IoTData>;
    fn validate_device(&self, device_id: &str, signature: &[u8]) -> Result<bool>;
}
```

### IoT Data Format

```rust
pub struct IoTData {
    pub device_id: String,
    pub timestamp: u64,
    pub data_type: String,       // "temperature", "humidity", etc.
    pub value: DataValue,
    pub unit: String,            // "celsius", "percent", etc.
    pub quality: f64,            // 0.0-1.0 (data quality score)
    pub metadata: HashMap<String, String>,
}

pub enum DataValue {
    Float(f64),
    Integer(i64),
    Boolean(bool),
    String(String),
    Binary(Vec<u8>),
    Struct(HashMap<String, DataValue>),
    Array(Vec<DataValue>),
}
```

## Edge Processing

The edge processing engine filters, transforms, and aggregates data before submission.

### Rules Engine

```rust
// Example rule: alert if temperature > 40
Rule {
    condition: Condition::GT("temperature", 40.0),
    action: ProcessingAction::CriticalAlert,
}

// Logical operators
Condition::AND(vec![
    Condition::GT("temperature", 35.0),
    Condition::LT("humidity", 20.0),
])
```

Operators: `EQ`, `NEQ`, `GT`, `LT`, `GTE`, `LTE`, `CONTAINS`, `IN`, `AND`, `OR`.

### Anomaly Detection

The edge processor runs anomaly detection on incoming data:
- Statistical outlier detection
- Pattern recognition
- ML model inference (local models)

Anomalies trigger automatic submission regardless of batch schedule.

### Data Aggregation

| Strategy | Description |
|----------|-------------|
| `Average` | Mean of values in window |
| `Sum` | Total of values |
| `Min` / `Max` | Extremes |
| `Median` | Middle value |
| `Percentile(p)` | P-th percentile |

## Batch Processing

Individual sensor readings are batched to reduce transaction costs:

```rust
pub struct BatchProcessor {
    pub batch_size: usize,       // max readings per batch
    pub max_delay_ms: u64,       // max time before forced submit
    pub compression: bool,       // compress batch payload
    pub aggregation: bool,       // aggregate before submit
}
```

### Batch Triggers

| Trigger | Description |
|---------|-------------|
| Size reached | `batch_size` readings accumulated |
| Max delay | `max_delay_ms` elapsed since first reading |
| Critical priority | Anomaly or critical alert detected |

Batching reduces blockchain transactions by up to **90%** compared to per-reading submission.

## Security

### Authentication

Three authentication methods, combinable:

| Method | Description |
|--------|-------------|
| **Certificate** | X.509 certificate chain validation |
| **Token** | API token with expiry |
| **Biometric** | Device biometric verification |

### Data Encryption

- **AES-256** encryption for data in transit and at rest
- **Integrity hashing** for tamper detection
- Certificate revocation checking

### Connector Configuration

```rust
pub struct ConnectorConfig {
    pub max_requests_per_second: u32,
    pub timeout_ms: u64,
    pub retry_attempts: u32,
}
```

## On-Chain Integration

IoT data is submitted as transactions with the IoT fee rate:

```rust
let tx = TransactionBuilder::new()
    .oracle_call(
        "iot_oracle_address",
        "submit_sensor_data",
        &batch_payload,
    )
    .nonce(nonce)
    .fee(50_000_000_000_000)  // 0.00005 SAVT (IoT rate)
    .build_and_sign(&device_wallet)?;
```

### Oracle Feed Integration

Submitted IoT data becomes available as oracle feeds:

```rust
// Request latest sensor reading
let tx = oracle.request_data(
    &iot_oracle_address,
    "temperature",
    b"sensor_001",
).await?;
```

### Connector Metrics

```rust
pub struct ConnectorMetrics {
    pub requests_served: u64,
    pub errors_count: u64,
    pub average_response_time: f64,
}
```
