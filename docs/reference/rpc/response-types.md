# RPC Response Types

Detailed JSON schemas for all RPC response objects.

## BlockResponse

Returned by `chain_getBlock`, `chain_getBlockByNumber`, `chain_getBlockByHash`, `chain_getLatestBlock`.

```json
{
  "hash": "0a1b2c...",
  "height": 42,
  "timestamp": 1710000000,
  "parent_hash": "ff0011...",
  "state_root": "abc123...",
  "tx_root": "def456...",
  "proposer": "aabbcc...",
  "version": 1,
  "transaction_count": 15
}
```

| Field | Type | Description |
|-------|------|-------------|
| `hash` | string | Block hash (hex, 128 chars = 64 bytes SHA-512) |
| `height` | u64 | Block number |
| `timestamp` | u64 | Unix timestamp (seconds) |
| `parent_hash` | string | Parent block hash (hex) |
| `state_root` | string | State root hash (hex, 64 chars) |
| `tx_root` | string | Transaction root hash (hex, 64 chars) |
| `proposer` | string | Proposer public key (hex, 64 chars) |
| `version` | u32 | Block format version |
| `transaction_count` | u64 | Number of transactions in the block |

## AccountResponse

Returned by `account_getAccount`, `account_getBalance`, `account_getNonce`.

```json
{
  "address": "aabbcc...",
  "balance": "1000000000000000000",
  "nonce": 5
}
```

| Field | Type | Description |
|-------|------|-------------|
| `address` | string | Account address (hex, 64 chars = 32-byte ed25519 pubkey) |
| `balance` | string | Balance in smallest unit (u128 as decimal string, 18 decimals) |
| `nonce` | u64 | Current transaction nonce |

## TransactionResponse

Returned by `tx_getTransaction`.

```json
{
  "hash": "aabbcc...",
  "from": "112233...",
  "to": "445566...",
  "amount": 1000,
  "nonce": 1,
  "fee": 1000000000000000,
  "timestamp": 1710000000,
  "block_height": 42
}
```

| Field | Type | Description |
|-------|------|-------------|
| `hash` | string | Transaction hash (hex) |
| `from` | string | Sender address (hex) |
| `to` | string | Recipient address (hex) |
| `amount` | u64 | Transfer amount |
| `nonce` | u64 | Sender nonce at time of submission |
| `fee` | u128? | Transaction fee (optional) |
| `timestamp` | u64? | Block timestamp if confirmed |
| `block_height` | u64? | Block height if confirmed |

## TransactionReceiptResponse

Returned by `tx_getTransactionReceipt`.

```json
{
  "hash": "aabbcc...",
  "from": "112233...",
  "to": "445566...",
  "amount": 1000,
  "fee": 1000000000000000,
  "block_height": 42,
  "block_hash": "ff0011...",
  "timestamp": 1710000000,
  "status": "confirmed"
}
```

Status values: `"confirmed"`, `"pending"`, `"not_found"`.

## HealthResponse

Returned by `savitri_health`.

```json
{
  "status": "ok",
  "service": "savitri-rpc",
  "mode": "lightnode"
}
```

Mode values: `"lightnode"`, `"masternode"`, `"unknown"`.

## PouLocalResponse

Returned by `pou_getConsensusState` / `savitri_pouLocal`.

```json
{
  "local_score": 750,
  "leader": "12Dxyz...",
  "leader_score": 920,
  "epoch": 15,
  "local_is_leader": false,
  "election_ready": true
}
```

| Field | Type | Description |
|-------|------|-------------|
| `local_score` | u16? | This node's PoU score (0-1000) |
| `leader` | string? | Current leader peer ID |
| `leader_score` | u16? | Leader's PoU score |
| `epoch` | u64? | Current epoch number |
| `local_is_leader` | bool | Whether this node is the current leader |
| `election_ready` | bool | Whether an election can run |

## PouPeersResponse

Returned by `savitri_pouPeers`.

```json
{
  "peers": {
    "12D3KooW...abc": 750,
    "12D3KooW...def": 820
  }
}
```

## NodeInfoResponse

Returned by `net_nodeInfo`.

```json
{
  "node_id": "12D3KooW...",
  "protocol_version": "1.0.0",
  "network": "savitri-testnet",
  "listening": true,
  "peer_count": 14,
  "block_height": 42,
  "syncing": false,
  "mode": "lightnode"
}
```

## MempoolSizeResponse

Returned by `mempool_getSize`.

```json
{
  "pending": 150,
  "queued": 30
}
```

## TokenInfoResponse

Returned by `token_getTokenInfo`.

```json
{
  "token_id": "savt",
  "name": "Savitri Test Token",
  "symbol": "TEST",
  "decimals": 18,
  "total_supply": "100000000000000000000000000"
}
```

## FaucetClaimResponse

Returned by `savitri_faucetClaim`.

```json
{
  "tx_hash": "0xabc...",
  "amount": "5000000000000000000"
}
```

Amount is 5 SAVT (5 * 10^18 smallest units).

## SyncingResponse

Returned by `savitri_syncing`.

```json
{
  "syncing": true,
  "current_block": 100,
  "highest_block": 500
}
```

## MonolithInfoResponse

Returned by `savitri_getMonolithHead`.

```json
{
  "exec_height": 1000,
  "window_start": 900,
  "epoch_id": 5,
  "block_count": 100,
  "size_bytes": 524288,
  "monolith_id": "mono_1000",
  "produced_at_ms": 1710000000000,
  "cosignature_count": 3
}
```
