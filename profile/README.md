<p align="center">
  <a href="https://github.com/savitri-network">
    <img src="[https://github.com/Savitri-Network/.github/blob/main/assets/savitri-banner.png]" alt="Savitri Network" width="100%"/>
  </a>
</p>

<h1 align="center">Blockchain that rewards behavior, not wealth</h1>

<p align="center">
  <strong>A modular L1 blockchain built in Rust with Proof-of-Unity — consensus that scores nodes<br/>on availability, integrity, and participation instead of stake or hashrate.</strong>
</p>

<p align="center">
  <a href="https://docs.savitrinetwork.com"><img src="https://img.shields.io/badge/Docs-docs.savitrinetwork.com-0f3460?style=for-the-badge" alt="Documentation"/></a>&nbsp;
  <a href="https://discord.gg/savitri"><img src="https://img.shields.io/badge/Discord-Join_Community-5865F2?style=for-the-badge&logo=discord&logoColor=fff" alt="Discord"/></a>&nbsp;
  <a href="https://twitter.com/savitri_network"><img src="https://img.shields.io/badge/X-Follow-000?style=for-the-badge&logo=x" alt="Twitter"/></a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Rust-000000?style=flat-square&logo=rust&logoColor=white" alt="Rust"/>
  <img src="https://img.shields.io/badge/libp2p-0055FF?style=flat-square&logo=libp2p&logoColor=white" alt="libp2p"/>
  <img src="https://img.shields.io/badge/RocksDB-2D2D2D?style=flat-square&logo=rocksdb&logoColor=white" alt="RocksDB"/>
  <img src="https://img.shields.io/badge/ed25519-4A154B?style=flat-square&logo=letsencrypt&logoColor=white" alt="ed25519"/>
  <img src="https://img.shields.io/badge/BLAKE3-1a1a2e?style=flat-square" alt="BLAKE3"/>
  <img src="https://img.shields.io/badge/ZKP_(Groth16/PLONK)-e94560?style=flat-square" alt="ZKP"/>
  <img src="https://img.shields.io/badge/Flutter-02569B?style=flat-square&logo=flutter&logoColor=white" alt="Flutter"/>
  <img src="https://img.shields.io/badge/Tauri-24C8D8?style=flat-square&logo=tauri&logoColor=white" alt="Tauri"/>
  <img src="https://img.shields.io/badge/React-61DAFB?style=flat-square&logo=react&logoColor=000" alt="React"/>
  <img src="https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white" alt="TypeScript"/>
  <img src="https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white" alt="Docker"/>
  <img src="https://img.shields.io/badge/Prometheus-E6522C?style=flat-square&logo=prometheus&logoColor=white" alt="Prometheus"/>
  <img src="https://img.shields.io/badge/Grafana-F46800?style=flat-square&logo=grafana&logoColor=white" alt="Grafana"/>
  <img src="https://img.shields.io/badge/SQLite-003B57?style=flat-square&logo=sqlite&logoColor=white" alt="SQLite"/>
  <img src="https://img.shields.io/badge/WASM-654FF0?style=flat-square&logo=webassembly&logoColor=white" alt="WASM"/>
  <img src="https://img.shields.io/badge/BFT_Consensus-0f3460?style=flat-square" alt="BFT"/>
  <img src="https://img.shields.io/badge/License-MIT/Apache_2.0-green?style=flat-square" alt="License"/>
</p>

---

### Why Proof-of-Unity?

| | **Proof-of-Work** | **Proof-of-Stake** | **Proof-of-Unity** |
|---|---|---|---|
| **Who wins?** | Biggest mining farm | Richest validator | Best-performing node |
| **Energy** | Nation-sized consumption | Efficient | Efficient |
| **Centralization** | Hardware cartels | Whale dominance | Meritocratic |
| **Finality** | ~60 min | ~12 sec | **150 ms** |

---

### The Numbers

<table>
<tr>
<td align="center"><strong>150 ms</strong><br/><sub>BFT Finality</sub></td>
<td align="center"><strong>50K TPS</strong><br/><sub>Max Throughput</sub></td>
<td align="center"><strong>$0</strong><br/><sub>Min Node Cost (phone)</sub></td>
<td align="center"><strong>2B Fixed</strong><br/><sub>$SAVI Supply</sub></td>
<td align="center"><strong>0%</strong><br/><sub>Team Unlock at TGE</sub></td>
</tr>
</table>

> Every token is hardcoded in the protocol. No wallet, multisig, or foundation controls the supply. **The code IS the tokenomics.**

---

### Repositories

| | Repository | What it does |
|---|---|---|
| ⚙️ | [**savitri-network**](https://github.com/savitri-network/savitri-network) | Monorepo — 13 Rust crates, full blockchain stack |
| 📱 | [**savitri-mobile**](https://github.com/savitri-network/savitri-network/tree/main/savitri-mobile) | Flutter wallet + node monitoring (iOS & Android) |
| 🛠️ | [**savitri-sdk**](https://github.com/savitri-network/savitri-network/tree/main/savitri-sdk) | Client library, wallet tools, CLI |
| 🧪 | [**savitri-testnet**](https://github.com/savitri-network/savitri-network/tree/main/savitri-testnet) | Docker testnet with Prometheus + Grafana |
| 📖 | [**docs**](https://github.com/savitri-network/docs) | Whitepaper, tokenomics, architecture, guides |

---

### Quick Start

```bash
git clone --recurse-submodules https://github.com/savitri-network/savitri-network.git
cd savitri-network
cargo build --release -p savitri-lightnode
./target/release/lightnode --listen-port 4001 --bootstrap <PEER_ID>@/ip4/<IP>/tcp/4002
```

> Runs on **4 GB RAM, any CPU**. No ASIC, no enterprise server, no six-figure stake.

---

### Built For

<table>
<tr>
<td align="center" width="33%">🌐 <strong>IoT</strong><br/><sub>Dedicated tx type at $0.000125<br/>50x cheaper than standard</sub></td>
<td align="center" width="33%">🧠 <strong>Federated AI</strong><br/><sub>On-chain ML training<br/>Data stays local</sub></td>
<td align="center" width="33%">👥 <strong>Everyone</strong><br/><sub>Run a node from your phone<br/>No minimum stake barrier</sub></td>
</tr>
</table>

---

<p align="center">
  <a href="https://docs.savitri.network">Read the Docs</a> · <a href="https://github.com/savitri-network/docs/blob/main/README.md">Whitepaper & Tokenomics</a> · <a href="https://github.com/savitri-network/savitri-network/blob/main/CONTRIBUTING.md">Contribute</a> · <a href="https://github.com/savitri-network/savitri-network/security">Security</a>
</p>

<p align="center">
  <sub>Built with Rust · Secured by Proof-of-Unity · Open Source (MIT + Apache 2.0)</sub>
</p>
