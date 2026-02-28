# IOTA Roadmap and Starfish Consensus

## Table of Contents

- [Overview](#overview)
- [1. Current State: Mysticeti Consensus](#1-current-state-mysticeti-consensus)
- [2. Starfish: Next-Generation Consensus](#2-starfish-next-generation-consensus)
- [3. How Starfish Works](#3-how-starfish-works)
- [4. Mysticeti vs Starfish Comparison](#4-mysticeti-vs-starfish-comparison)
- [5. Impact for Developers](#5-impact-for-developers)
- [6. Starfish Development Timeline](#6-starfish-development-timeline)
- [7. Other 2026 Protocol Improvements](#7-other-2026-protocol-improvements)
- [8. Key Ecosystem Milestones](#8-key-ecosystem-milestones)
- [References](#references)

## Overview

IOTA Rebased launched in Q2 2025 with Mysticeti consensus. The next major protocol evolution is **Starfish**, a next-generation BFT consensus protocol designed for high security, low latency, and efficient communication — even under adverse network conditions. Starfish has been deployed on Devnet (Q3 2025) and is planned for Mainnet deployment.

## 1. Current State: Mysticeti Consensus

Mysticeti is IOTA's current consensus mechanism, inherited from the Sui fork:

- **Type:** DAG-based Byzantine Fault Tolerant (BFT) consensus
- **Performance:** 50,000+ TPS with ~400ms average finality
- **Validators:** Up to 100 elected validators via DPoS
- **Parallel execution:** Owned objects bypass consensus entirely (fast path)
- **Consensus needed only for:** Shared objects (sequentialized)

Mysticeti works well, but Starfish improves on it in several key areas.

## 2. Starfish: Next-Generation Consensus

Starfish is an IOTA-developed consensus protocol that will replace Mysticeti. It was introduced in the academic paper:

> **"Starfish: A DAG-Based BFT Consensus with Optimistic Fairness and Efficient Communication"**
> Authors: Nikita Polyanskii, Sebastian Mueller (Aix-Marseille University), Ilya Vorobyev (IOTA Foundation)
> Published: [IACR ePrint 2025/567](https://eprint.iacr.org/2025/567) ([PDF](https://eprint.iacr.org/2025/567.pdf))

### Key Innovations

1. **Encoded Cordial Dissemination** — Combines Reed-Solomon erasure coding with Data Availability Certificates (DACs):
   - Each validator distributes complete data of its own blocks
   - Other validators' blocks are distributed as encoded shards
   - Only **f+1 shards** needed to reconstruct data (from n=3f+1 validators)
   - Result: **O(n) amortized communication** per byte of transaction data

2. **High Security Under Degradation** — Provably high security and low latency even when the network degrades (Byzantine validators, network partitions)

3. **Performance:**
   - Up to **150,000 TPS** in experimental tests
   - Average latency: **7.5δ** (where δ = actual network delay)
   - Worst-case latency: **11δ**
   - Works in the **partially synchronous** model

4. **Linear Communication Complexity** — Amortized O(n) per transaction byte, making it efficient for large validator sets

5. **Fairer Validator Committee Selection** — Dynamic selection based on performance, preventing stake centralization

## 3. How Starfish Works

### Erasure Coding Approach

In traditional BFT consensus, every validator broadcasts the full block to every other validator — O(n²) communication. Starfish changes this:

```
Traditional (Mysticeti):
  Validator A → broadcasts full block to ALL validators
  Total data sent: O(n × block_size) per validator

Starfish:
  Validator A → sends own full block data
                + encoded SHARDS of other blocks
  Each shard = block_size / (f+1)
  Total data sent: O(block_size) amortized per validator
```

### Data Availability Certificates (DACs)

Before a block is committed, validators collect **Data Availability Certificates** — cryptographic proofs that enough shards exist in the network to reconstruct any block. This guarantees that even if some validators go offline, data can always be recovered.

### Partially Synchronous Model

Starfish operates in the partially synchronous model:
- **Safety** is guaranteed at all times (even during network partitions)
- **Liveness** is guaranteed after the network stabilizes (Global Stabilization Time)
- This is the same model used by most production BFT protocols (PBFT, HotStuff, Mysticeti)

## 4. Mysticeti vs Starfish Comparison

| Feature | Mysticeti (Current) | Starfish (Planned) |
|---------|--------------------|--------------------|
| Type | DAG-based BFT | DAG-based BFT with erasure coding |
| Communication | O(n²) per block | O(n) amortized per block |
| TPS (tested) | ~50,000+ | ~150,000 |
| Average finality | ~400ms | Comparable or better |
| Latency (theoretical) | — | 7.5δ average, 11δ worst case |
| Network degradation | Performance drops | Maintains security + low latency |
| Validator fairness | Stake-weighted | Dynamic performance-based selection |
| Data availability | Full replication | Erasure-coded shards + DACs |
| Academic paper | MystenLabs (Sui) | [IACR 2025/567](https://eprint.iacr.org/2025/567) |

## 5. Impact for Developers

### What Changes for Your Code?

**Nothing.** Starfish is a consensus-layer upgrade — it does not change:

- The Move language or VM
- The object model
- Transaction format or PTBs
- SDK APIs or RPC endpoints
- How you write, test, or deploy smart contracts

### What You'll Notice

- **Higher throughput** — more transactions per second on shared objects
- **Lower latency** — faster finality for consensus-dependent operations
- **Better reliability** — network stays performant under stress/attacks
- **Same gas costs** — consensus efficiency doesn't change gas pricing (separate mechanism)

### When Starfish Goes Live

Your deployed contracts will work identically. No redeployment needed. The upgrade happens at the protocol level, transparent to smart contracts and applications.

## 6. Starfish Development Timeline

| Date | Milestone |
|------|-----------|
| **2025 Q1** | Starfish paper published ([ePrint 2025/567](https://eprint.iacr.org/2025/567)) |
| **2025 Q2** | Active development, presented at Protocol Berg v2 |
| **2025 Q3** | Non-sharded version deployed to **Devnet** (alternating with Mysticeti for comparison) |
| **2025 Q4** | Sharded version (with erasure coding) targeted for Devnet |
| **2026** | Continued refinement, Testnet deployment, Mainnet deployment (TBD) |

> **Note:** The non-sharded version broadcasts complete transaction data. The full version uses erasure-coded shards for the O(n) communication advantage. The codebase is substantially complete; the primary remaining component is the decoding portion.

## 7. Other 2026 Protocol Improvements

Based on the [IOTA 2025 Year Review](https://blog.iota.org/iota-2025-review/) and quarterly progress reports:

### Local Gas Fee Markets
- Per-object or per-dApp gas pricing
- Congested dApps pay more without affecting the rest of the network
- Fairer, more efficient network economics

### IOTA Names
- Human-readable addresses (e.g., `dom.iota` instead of `0x3d33f...`)
- Released on Testnet in Q3 2025 (https://testnet.iotanames.com/)
- Mainnet deployment in evaluation

### Validator Reputation Score
- Deployed on Mainnet (each node tracks scores)
- Being brought through consensus to become part of system state
- Will enable performance-based staking reward distributions

### Account Abstraction
- Onchain objects functioning as accounts with dynamic authentication
- Enables multi-sig solutions (like Gnosis Safe)
- Social recovery and advanced fund management
- Core version deployed to Devnet for demo applications

### Move View Functions
- Easier access to detailed data of onchain objects
- Improved developer experience for state queries
- Shipped to Testnet with SDK integrations

### SDK v2
- Maintained separately from monorepo (faster compilation)
- New bindings: Go, Kotlin/Java, Python, WASM
- Beta version targeted

### TWIN on Mainnet
- Real trade transactions anchored on IOTA Mainnet since January 2026
- Three countries confirmed for ADAPT deployment, five more piloting
- See [twin.md](twin.md) for details

### Enhanced Sequencing Algorithm
- Reduces latency and increases throughput
- Intelligent ordering of transaction execution
- Already deployed improvements in 2025

## 8. Key Ecosystem Milestones

| Year | Milestone |
|------|-----------|
| 2024 | IOTA Rebased announced, development begins (Sui fork) |
| 2025 Q2 | **IOTA Rebased Mainnet launch** — fully decentralized L1, Move smart contracts |
| 2025 Q2 | TWIN Foundation officially launched |
| 2025 Q3 | Starfish non-sharded on Devnet; IOTA Names on Testnet |
| 2025 Q3 | IOTA Notarization and Hierarchies released |
| 2025 | IOTA turns 10; 50,000+ TPS, ~400ms finality achieved |
| 2025 | BitGo custody, LayerZero/Stargate cross-chain, Uphold listing |
| 2026 | Starfish full deployment; TWIN Mainnet; ADAPT country rollouts |
| 2026 | Local gas fee markets; Account abstraction; IOTA Names Mainnet |

## References

- [Starfish Paper — IACR ePrint 2025/567](https://eprint.iacr.org/2025/567) ([PDF](https://eprint.iacr.org/2025/567.pdf))
- [IOTA 2025 Year Review](https://blog.iota.org/iota-2025-review/)
- [Q3 2025 Progress Update](https://blog.iota.org/q3-2025-progress-update/)
- [Q2 2025 Progress Report](https://blog.iota.org/q2-2025-progress-report/)
- [IOTA Rebased launch blog](https://blog.iota.org/builders-welcome-rebase-complete/)
