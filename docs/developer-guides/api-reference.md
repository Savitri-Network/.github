# RPC API Reference

Savitri Network exposes a **JSON-RPC 2.0** API over HTTP. All requests are sent as `POST` to `/` or `/rpc`.

## Connection

- **Default endpoint**: `http://127.0.0.1:8545/rpc`
- **Protocol**: JSON-RPC 2.0 (no REST endpoints)
- **TLS**: Not terminated by the RPC server. Use a reverse proxy (nginx, Caddy) for production.
- **CORS**: Restricted to `localhost` and `127.0.0.1` origins by default.

## Rate Limiting

| Limit | Value |
|-------|-------|
| Global | 200 requests/second |
| Per-IP | 50 requests/second |
| Max batch size | 100 requests |
| Max body size | 1 MB |

Exceeding limits returns HTTP 429 (Too Many Requests).

## Request Format

```json
{
  "jsonrpc": "2.0",
  "method": "method_name",
  "params": [],
  "id": 1
}
```

Parameters can be passed as positional array or named object.

## Batch Requests

Send an array of requests to process multiple calls in a single HTTP request:

```json
[
  {"jsonrpc": "2.0", "method": "savitri_blockNumber", "params": [], "id": 1},
  {"jsonrpc": "2.0", "method": "savitri_health", "params": [], "id": 2}
]
```

Maximum 100 requests per batch.

## Error Codes

| Code | Meaning |
|------|---------|
| -32700 | Parse error (invalid JSON) |
| -32600 | Invalid request (not JSON-RPC 2.0) |
| -32601 | Method not found |
| -32602 | Invalid params |
| -32603 | Internal server error |
| -32001 | Resource not found |
| -32002 | Service unavailable (storage/mempool) |
| -32003 | Not implemented |

## Method Reference

### Chain Methods

| Method | Params | Description |
|--------|--------|-------------|
| `chain_getBlockHeight` | `[]` | Get current block height |
| `savitri_blockNumber` | `[]` | Alias for chain_getBlockHeight |
| `chain_getBlock` | `[height]` | Get block by height |
| `savitri_getBlockByHeight` | `[height]` | Alias for chain_getBlock |
| `chain_getBlockByNumber` | `[height]` | Get block by height |
| `chain_getBlockByHash` | `[hash]` | Get block by hash (scans last 1000 blocks) |
| `chain_getLatestBlock` | `[]` | Get the most recent block |
| `chain_getChainInfo` | `[]` | Get chain metadata (id, name, height, version) |
| `savitri_getBlockHash` | `[height]` | Get block hash at height |

### Transaction Methods

| Method | Params | Description |
|--------|--------|-------------|
| `tx_sendTransaction` | `[raw_tx_hex]` | Submit signed transaction |
| `savitri_sendRawTransaction` | `[raw_tx_hex]` | Alias for tx_sendTransaction |
| `tx_getTransaction` | `[hash]` | Get transaction by hash |
| `savitri_getTransaction` | `[hash]` | Alias for tx_getTransaction |
| `tx_getTransactionReceipt` | `[hash]` | Get transaction receipt with status |
| `tx_getTransactionsByBlock` | `[height]` | Get all transactions in a block |
| `tx_getPendingTransactions` | `[]` | List pending mempool transactions |

### Account Methods

| Method | Params | Description |
|--------|--------|-------------|
| `account_getAccount` | `[address]` | Get balance and nonce |
| `savitri_getAccount` | `[address]` | Alias for account_getAccount |
| `account_getBalance` | `[address]` | Get balance only |
| `account_getNonce` | `[address]` | Get nonce only |
| `account_getTokenBalance` | `[address, token_id]` | Get token balance |

### Network Methods

| Method | Params | Description |
|--------|--------|-------------|
| `net_version` | `[]` | Get network version |
| `net_peerCount` | `[]` | Get connected peer count |
| `net_listening` | `[]` | Check if node is listening |
| `net_peers` | `[]` | List connected peers with scores |
| `net_nodeInfo` | `[]` | Get node info (id, mode, peers, height) |

### Mempool Methods

| Method | Params | Description |
|--------|--------|-------------|
| `mempool_getSize` | `[]` | Get pending and queued counts |
| `mempool_getPendingTransactions` | `[]` | List pending transactions |
| `mempool_getTransactionStatus` | `[hash]` | Check if transaction is in mempool |

### Consensus / PoU Methods

| Method | Params | Description |
|--------|--------|-------------|
| `pou_getConsensusState` | `[]` | Get local PoU score, leader, epoch |
| `savitri_pouLocal` | `[]` | Alias for pou_getConsensusState |
| `savitri_pouPeers` | `[]` | Get all peer PoU scores |
| `pou_getValidators` | `[]` | List validators with scores |
| `pou_getStakeInfo` | `[address]` | Get staking info for address |
| `pou_getEpochInfo` | `[]` | Get current epoch details |
| `savitri_pouGroups` | `[]` | List PoU groups (masternode only) |
| `savitri_pouMasternodes` | `[]` | List masternodes (masternode only) |
| `savitri_pouGroupNodes` | `[group_id]` | List nodes in a group (masternode only) |

### Token Methods

| Method | Params | Description |
|--------|--------|-------------|
| `token_getTokenInfo` | `[token_id]` | Get token metadata (name, symbol, decimals, supply) |
| `token_getTokenBalance` | `[address, token_id]` | Get token balance |
| `token_getTokenTransfers` | `[token_id]` | Get transfer history |

### Utility Methods

| Method | Params | Description |
|--------|--------|-------------|
| `savitri_health` | `[]` | Health check (status, service, mode) |
| `savitri_protocolVersion` | `[]` | Get protocol version |
| `savitri_syncing` | `[]` | Get sync status (current/highest block) |
| `savitri_gasPrice` | `[]` | Get current gas price |
| `savitri_estimateGas` | `[]` | Estimate gas for transaction |
| `savitri_faucetClaim` | `[address]` | Claim testnet tokens (5 SAVT) |

### Monolith Methods

| Method | Params | Description |
|--------|--------|-------------|
| `savitri_getMonolithHead` | `[]` | Get latest monolith metadata |
| `savitri_getMonolithsForRange` | `[start, end]` | Get monoliths in height range |
| `savitri_getMonolith` | `[monolith_id]` | Get full monolith block |

## Response Types

See [Response Types](response-types.md) for detailed JSON schemas of each response object.
