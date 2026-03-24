# Storage Architecture

Savitri Network uses **RocksDB** as its primary storage backend with column families for data isolation and efficient access patterns.

## Column Families

| Column Family | Key | Value | Purpose |
|--------------|-----|-------|---------|
| `CF_BLOCKS` | height (u64 LE) | Block (bincode) | Block storage |
| `CF_TRANSACTIONS` | tx_hash (32 bytes) | Transaction (bincode) | Transaction lookup |
| `CF_ACCOUNTS` | address (32 bytes) | Account (24 bytes) | Balance + nonce |
| `CF_METADATA` | string key | varies | Chain head, config |
| `CF_RECEIPTS` | tx_hash | Receipt (bincode) | Transaction receipts |
| `CF_CONTRACTS` | address | ContractInfo | Smart contract state |
| `CF_GOVERNANCE` | proposal_id | Proposal (bincode) | Governance proposals |
| `CF_ORACLE` | feed_id | OracleData | Oracle feeds |
| `CF_BONDS` | address | Bond amount | Validator bonds |
| `CF_VOTE_TOKENS` | address | VoteToken balance | Governance voting tokens |
| `CF_TREASURY` | key | Treasury state | Network treasury |
| `CF_VESTING` | schedule_id | VestingSchedule | Token vesting |
| `CF_REWARD_COINS` | address | Reward balance | Node rewards |
| `CF_FEE_METRICS` | key | Fee data | Fee tracking |
| `CF_SUPPLY_METRICS` | key | Supply data | Token supply |
| `CF_ACTIVE_NODES` | node_id | Activity | Node liveness |
| `CF_MONOLITHS` | monolith_id | Monolith | Block compression |
| `CF_FL` | key | FL data | Federated learning |
| `CF_POU` | node_id | PoU score | Consensus scores |

## Account Encoding

Accounts use a compact 24-byte fixed-width encoding:

```
Bytes 0-15:  balance (u128, little-endian)
Bytes 16-23: nonce   (u64, little-endian)
```

Backward compatible: old 16-byte format (balance only, nonce=0) is still supported.

Empty accounts (balance=0, nonce=0) are **never persisted** to save storage space.

## State Root

The state root is computed via lexicographic database snapshot:

```
seed  = H("STATEv1-LE")
leaf  = H("STATE" || key || value)  for each key-value pair
root  = rolling_accumulate(seed, leaf_1, leaf_2, ..., leaf_n)
```

Keys are iterated in lexicographic order for determinism.

## Storage Trait

All storage backends implement the `StorageTrait`:

```rust
pub trait StorageTrait: Send + Sync {
    fn get_cf(&self, cf: &str, key: &[u8]) -> Result<Option<Vec<u8>>>;
    fn put_cf(&self, cf: &str, key: &[u8], value: &[u8]) -> Result<()>;
    fn delete_cf(&self, cf: &str, key: &[u8]) -> Result<()>;
    fn get_cf_prefix(&self, cf: &str, prefix: &[u8]) -> Result<Vec<(Vec<u8>, Vec<u8>)>>;
    // ... batch operations, iteration, etc.
}
```

## Safe Deserialization

All bincode deserialization uses a maximum size limit to prevent memory exhaustion:

```rust
const MAX_BINCODE_SIZE: u64 = 16 * 1024 * 1024; // 16MB
```

## Caching

LRU caching layer for frequently accessed data:

- Account cache: reduces RocksDB reads for balance/nonce checks
- Contract storage cache: caches SLOAD results within contract execution
- Score cache: LRU cache for mempool TX scoring (avoids recomputation)

## RocksDB Configuration

| Parameter | Development | Production |
|-----------|-------------|------------|
| Cache size | 256 MB | 1024 MB |
| Write buffer | 16 MB | 64 MB |
| Max open files | 100 | 1000 |
| Sync interval | 5s | 30s |
| Compression | Disabled | Enabled |

## Batch Operations

Atomic batch writes ensure consistency:

```rust
let mut batch = storage.batch();
batch.put_cf(CF_ACCOUNTS, &addr, &account.encode());
batch.put_cf(CF_TRANSACTIONS, &tx_hash, &tx_bytes);
batch.put_cf(CF_BLOCKS, &height_key, &block_bytes);
batch.commit()?;
```

## Vesting System

Token vesting schedules support three types:

| Type | Behavior |
|------|----------|
| `Linear` | Tokens released linearly over duration |
| `Cliff` | No tokens before cliff period, then linear |
| `Staged` | Genesis compatibility mode |

## Feature Flags

| Flag | Description |
|------|-------------|
| `rocksdb` | RocksDB backend (default) |
| `memory` | In-memory backend (testing) |
| `prometheus` | Prometheus metrics export |
| `tempfile` | Temporary storage for tests |
