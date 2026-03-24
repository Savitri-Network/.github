# Security Policy

## Reporting Vulnerabilities

If you discover a security vulnerability in Savitri Network, please report it responsibly.

**Do NOT open a public GitHub issue for security vulnerabilities.**

### How to Report

1. Email: security@savitri.network
2. Include:
   - Description of the vulnerability
   - Steps to reproduce
   - Potential impact assessment
   - Suggested fix (if any)

### Response Timeline

| Stage | Timeframe |
|---|---|
| Acknowledgment | Within 48 hours |
| Initial assessment | Within 5 business days |
| Fix development | Depends on severity |
| Public disclosure | After fix is deployed |

### Scope

In scope:
- savitri-core, savitri-consensus, savitri-p2p, savitri-storage
- savitri-mempool, savitri-contracts, savitri-rpc
- savitri-masternode, savitri-lightnode, savitri-guardian
- savitri-zkp, savitri-sdk
- savitri-mobile (Rust FFI layer)
- savitri-game-api

Out of scope:
- Third-party dependencies (report upstream)
- Social engineering attacks
- Denial of service via expected rate limiting

### Severity Classification

| Severity | Description | Examples |
|---|---|---|
| Critical | Network consensus failure, fund theft | Double-spend, signature bypass, private key leak |
| High | Data loss, unauthorized access | Storage corruption, RPC auth bypass |
| Medium | Degraded service, information leak | Mempool manipulation, peer data exposure |
| Low | Minor issues, edge cases | Non-exploitable crashes, log leaks |

## Security Architecture

See [Threat Model](threat-model.md) for the complete security analysis including:
- Cryptographic primitives (ed25519, blake3, SHA-512)
- Network security (noise protocol, gossipsub validation)
- Consensus security (BFT 2/3 quorum, slashing)
- Smart contract sandboxing
- Input validation and rate limiting
