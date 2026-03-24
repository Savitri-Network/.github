# Savitri Network — Documentation

Technical documentation for the Savitri Network blockchain platform.

---

## Contents

### [Whitepaper](whitepaper/)
- Savitri Whitepaper (PDF) — *coming soon*

### [Tokenomics](tokenomics/)
- [Token Allocation](tokenomics/allocation.md) — $SAVI distribution, vesting schedules, TGE unlock
- [Emission Schedule](tokenomics/emission-schedule.md) — Halving phases, staking pool, 50-year projection
- [Fee Model](tokenomics/fee-model.md) — Transaction fees, IoT/AI fees, fee distribution
- [Deflationary Model](tokenomics/deflationary-model.md) — Burn mechanisms, net deflation timeline

### [Architecture](architecture/)
- [Overview](architecture/overview.md) — System architecture, crate dependency map
- [Consensus: Proof-of-Unity](architecture/consensus-pou.md) — PoU scoring, BFT voting, group formation
- [Network Topology](architecture/network-topology.md) — P2P stack, gossipsub, Kademlia, NAT traversal
- [Storage Model](architecture/storage-model.md) — RocksDB, column families, state roots
- [Mempool](architecture/mempool.md) — SIMD scoring, admission control, fair batching
- [Transaction Lifecycle](architecture/transaction-lifecycle.md) — From creation to finality
- [Block Production](architecture/block-production.md) — BFT rounds, fault tolerance, finality
- [State Machine](architecture/state-machine.md) — Core types, account model, state transitions
- [ZKP Integration](architecture/zkp-integration.md) — Pluggable backends: Groth16, PLONK, mock
- [IoT Transaction Type](architecture/iot-transaction-type.md) — Dedicated IoT TX, micro-fees, sensor management
- [Federated Learning](architecture/federated-learning.md) — FL contracts, decentralized training, incentives
- [Node Types](architecture/node-types.md) — Masternode vs Lightnode vs Guardian
- [Cryptography](architecture/cryptography.md) — ed25519, blake3, SHA-512, BIP-39/44

### [Developer Guides](developer-guides/)
- [Quickstart](developer-guides/quickstart.md) — Get running in 5 minutes
- [Run a Node](developer-guides/run-a-node.md) — Installation, configuration, all platforms
- [Smart Contracts](developer-guides/smart-contracts.md) — SAVITRI-20/721/1155 token standards
- [SDK Reference](developer-guides/sdk-reference.md) — Client library, wallet, CLI tools
- [API Reference](developer-guides/api-reference.md) — RPC & REST endpoints
- [Testnet Setup](developer-guides/testnet-setup.md) — Configuration, Docker Compose, Grafana
- [Token Standards](developer-guides/token-standards.md) — Creating and deploying tokens

### [API & SDK Reference](reference/)
- [SDK: RPC Client](reference/sdk/rpc-client.md) — JSON-RPC client usage
- [SDK: Wallet](reference/sdk/wallet.md) — Wallet creation, key management
- [SDK: Transactions](reference/sdk/transactions.md) — Building and signing transactions
- [SDK: Contracts](reference/sdk/contracts.md) — Contract interaction via SDK
- [SDK: Light Client](reference/sdk/light-client.md) — Lightweight verification
- [RPC: Response Types](reference/rpc/response-types.md) — All response type definitions
- [RPC: Examples](reference/rpc/examples.md) — Curl and SDK usage examples
- [Smart Contracts: SAVITRI-20](reference/smart-contracts/savitri-20.md) — Fungible token standard
- [Smart Contracts: SAVITRI-721](reference/smart-contracts/savitri-721.md) — Non-fungible token standard
- [Smart Contracts: SAVITRI-1155](reference/smart-contracts/savitri-1155.md) — Multi-asset standard
- [Smart Contracts: Oracle](reference/smart-contracts/oracle.md) — Oracle integration
- [Smart Contracts: Runtime](reference/smart-contracts/runtime.md) — Contract execution engine
- [Mobile Architecture](reference/mobile-architecture.md) — Flutter app, Rust FFI, BIP-39/44

### [Governance](governance/)
- [Governance Framework](governance/GOVERNANCE.md) — DAO structure, decision process
- [Voting Mechanism](governance/voting-mechanism.md) — $VOTE token, stake-weighted voting
- [Proposal Process](governance/proposal-process.md) — How to submit and vote on proposals

### [Security](security/)
- [Security Policy](security/SECURITY.md) — Responsible disclosure, reporting vulnerabilities
- [Threat Model](security/threat-model.md) — Attack vectors, cryptographic security, mitigations
- [Audits](security/audits/) — Third-party audit reports

### [Brand Assets](brand-assets/)
- [Logos](brand-assets/logos/) — SVG, PNG, dark/light variants
- [Colors](brand-assets/colors.md) — Official color palette, hex/RGB/CMYK
- [Brand Guidelines](brand-assets/guidelines.md) — Typography, tone of voice, usage rules

---

## Quick Links

| Resource | Link |
|---|---|
| Source Code | [github.com/Savitri-Network](https://github.com/Savitri-Network) |
| Report a Bug | [Issues](https://github.com/savitri-network/savitri-network/issues) |
| Security | [security/SECURITY.md](security/SECURITY.md) |

---

*Savitri Network — Built with Rust, Secured by Proof-of-Unity*
