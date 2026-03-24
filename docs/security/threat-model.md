# Security Model

## Cryptographic Primitives

| Primitive | Algorithm | Usage |
|-----------|-----------|-------|
| Signatures | Ed25519 (ed25519-dalek) | Transaction signing, block signing |
| Block hashing | SHA-512 | Block hash with domain tag `"BLK"` |
| Transaction hashing | SHA-256 | Signing message hash |
| State hashing | BLAKE3 | State root computation |
| Storage hashing | Keccak256 | Contract storage slot derivation |
| Compression | snap, lz4 | P2P message compression |

## Transport Security

### P2P Layer

- **Encryption**: libp2p Noise protocol (authenticated encryption)
- **Multiplexing**: Yamux
- **Identity**: Ed25519 peer keys
- **Max transmit size**: 256KB per message

### RPC Layer

- **TLS**: Not terminated by RPC server. Production deployments must use a TLS-terminating reverse proxy.
- **CORS**: Restricted to localhost origins. Configure allowed origins for production.
- **Body limit**: 1MB maximum request body.

## Rate Limiting

| Layer | Limit | Window |
|-------|-------|--------|
| Global RPC | 200 req/s | 1 second sliding window |
| Per-IP RPC | 50 req/s | 1 second sliding window |
| Batch size | 100 requests | Per HTTP request |
| Block hash scan | 1000 blocks | Per `chain_getBlockByHash` call |

## Input Validation

### RPC

- JSON-RPC version must be exactly `"2.0"`
- Error messages are sanitized before returning to clients
- Internal errors return generic messages (no stack traces or internal formats)
- Mempool rejection details are stripped from error responses

### Mempool

- **Nonce validation**: Sequential nonces required (gap tolerance up to 5000 for burst recovery)
- **Balance check**: Sender must have sufficient balance for amount + fee
- **Signature verification**: Ed25519 signature verified before admission
- **Quota system**: Per-class limits (Financial: 50K, IoT: 100K, Global: 100K)
- **Per-sender cap**: Maximum 512 pending transactions per sender

### Transaction Pipeline

Stateless guards (run first):
1. Signature format validation
2. Ed25519 signature verification
3. Public key → address derivation check

Stateful guards (run after stateless pass):
1. Nonce >= account nonce
2. Balance >= amount + fee
3. Account not blacklisted

Atomic commit: balance deduction and nonce increment happen atomically.

## Consensus Security

### BFT Quorum

- **Quorum**: 2f+1 (67%) of validators must sign Block Acceptance Certificate
- **Byzantine tolerance**: Up to f = (n-1)/3 byzantine nodes
- **Certificate verification**: Masternodes verify all attestation signatures

### PoU Anti-Gaming

- **Score smoothing**: Exponential moving average prevents score manipulation via transient behavior
- **Proposer rotation**: After 50 blocks, proposer must step down (prevents monopoly)
- **Minimum score threshold**: Score < 300 disqualifies from rewards

### Election Safety

- **Deterministic group IDs**: Based on epoch, not wall-clock time (prevents MN disagreement)
- **Election deduplication**: Only first result per round is committed
- **Soft-reject unknown groups**: Masternodes warn but continue processing for unknown group IDs

## Key Management

### SDK Wallet

- **Zeroization**: Private keys are zeroized on `Drop` (via `zeroize` crate)
- **No persistence**: SDK wallet does not save keys to disk
- **HTTPS enforcement**: Remote RPC endpoints require HTTPS by default

### Faucet

- **Round-robin keys**: 10 keypairs rotated to prevent nonce contention
- **Serialized claims**: `faucet_lock` mutex prevents TOCTOU race conditions
- **Claim limits**: 5 SAVT per claim, 24-hour cooldown per address

## Node Security

### Peer Liveness

- **Gossipsub keepalive**: PoU score sharing publishes via gossipsub to refresh `last_seen`
- **Inactivity timeout**: 600 seconds (configurable) before peer removal
- **SlowPeer disconnect**: Nodes disconnected if gossipsub send queue is saturated

### Resource Limits

- **Connection handler queue**: 50,000 messages (LN + MN)
- **Gossipsub mesh**: mesh_n=8, mesh_n_high=12, mesh_n_low=4
- **Stack size**: 8MB to prevent stack overflow
- **Memory**: Bounded mempool pools with backpressure

## Responsible Disclosure

Report security vulnerabilities via the project's responsible disclosure process. Do not file public issues for security bugs.
