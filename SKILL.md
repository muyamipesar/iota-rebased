---
name: iota-rebased
description: Complete developer guide for IOTA Rebased — the Move VM-powered L1 blockchain. Covers architecture, smart contracts, SDKs, migration from Stardust, tokenomics, and ecosystem. Use this skill when questions involve IOTA development, Move on IOTA, IOTA smart contracts, IOTA Rebased architecture, or migrating from legacy IOTA.
---

# IOTA Rebased — Developer Skill

## ⚠️ Critical Context

**IOTA Rebased is a completely different protocol from "old IOTA" (Tangle/Stardust/IOTA 2.0).**

- Old IOTA: DAG-based (Tangle), UTXO model, feeless, no L1 smart contracts
- **IOTA Rebased**: Object-based ledger, Move VM, DPoS consensus (Mysticeti), transaction fees, L1 smart contracts

If you encounter old documentation mentioning Tangle, Coordinator, feeless transactions, UTXO, or IOTA 2.0 — **it does NOT apply to IOTA Rebased**.

→ See [references/old-vs-new.md](references/old-vs-new.md) for a detailed comparison.

## What is IOTA Rebased?

IOTA Rebased is a Layer 1 smart contract platform using:

- **Move VM** for L1 smart contracts (resource-oriented, statically verified)
- **Object Model** instead of UTXO (objects can be owned, shared, or immutable)
- **Mysticeti Consensus** with Delegated Proof of Stake (DPoS)
- **Low transaction fees** (~0.005 IOTA per tx) that are burned
- **Staking rewards** (~767k IOTA minted per epoch, ~6% first-year inflation)
- **Sponsored transactions** (apps can pay fees on behalf of users)
- **Parallel transaction execution** for independent objects

Based on Sui's protocol with IOTA-specific improvements (resilient consensus, fairer gas pricing, dynamic validator selection).

## Quick Start

### 1. Install IOTA CLI

```bash
cargo install --locked --git https://github.com/iotaledger/iota.git --branch mainnet iota
```

Verify: `which iota`

### 2. Set Up Environment

```bash
# Connect to testnet
iota client new-env --alias testnet --rpc https://api.testnet.iota.cafe
iota client switch --env testnet

# Get your address
iota client active-address

# Get test tokens
# Visit: https://faucet.testnet.iota.cafe
```

### 3. Create Your First Move Package

```bash
iota move new my_first_package
cd my_first_package
```

### 4. Build and Test

```bash
iota move build
iota move test
iota move test --coverage
```

### 5. Publish

```bash
iota client publish --gas-budget 100000000
```

## Networks

| Network | RPC URL | Explorer | Faucet |
|---------|---------|----------|--------|
| **Mainnet** | `https://api.mainnet.iota.cafe` | [explorer.iota.org](https://explorer.iota.org/) | — |
| **Testnet** | `https://api.testnet.iota.cafe` | [explorer (testnet)](https://explorer.iota.org/?network=testnet) | [faucet](https://faucet.testnet.iota.cafe) |
| **Devnet** | `https://api.devnet.iota.cafe` | [explorer (devnet)](https://explorer.rebased.iota.org/?network=devnet) | [faucet](https://faucet.devnet.iota.cafe) |
| **Localnet** | `http://127.0.0.1:9000` | [local explorer](https://explorer.rebased.iota.org/?network=http://127.0.0.1:9000) | `http://127.0.0.1:9123/gas` |

GraphQL endpoints: replace `api.` with `graphql.` in RPC URLs.

Third-party RPC: [Ankr](https://rpc.ankr.com/iota_mainnet), [Monochain](https://rpc.mainnet.iota.monochain.p2p.org)

## Developer Workflow

```
Write Move code → Build (iota move build) → Test (iota move test)
→ Deploy to Testnet → Interact via CLI/SDK → Deploy to Mainnet
```

## Key Concepts for Developers

1. **Objects** are the fundamental unit (not UTXOs). Every asset is an object with a unique ID.
2. **Owned objects** can be used in parallel; **shared objects** require consensus ordering.
3. **Programmable Transaction Blocks (PTBs)** let you batch multiple operations in one tx.
4. **Move 2024 edition** adds method syntax (`v.push_back(x)`), enums, macros, index syntax.
5. **Max object size**: 250KB. Use dynamic fields for large/growing collections.
6. **Return objects** from functions instead of self-transferring — enables composability.

## Move vs EVM on IOTA

| | Move (L1) | EVM/Solidity (L2 - ISC) |
|---|-----------|------------------------|
| Layer | Layer 1 (native) | Layer 2 (IOTA Smart Contracts) |
| Performance | Higher (parallel execution) | Lower (sequential + L2 overhead) |
| Security | Static verification, resource types | Standard EVM security model |
| Use when | New projects, DeFi, NFTs, native assets | Porting existing Solidity code |

→ See [references/evm-layer2.md](references/evm-layer2.md)

## Reference Files

### Fundamentals

| File | Content |
|------|---------|
| [architecture.md](references/architecture.md) | Move VM, Object Model, Mysticeti consensus, DPoS |
| [old-vs-new.md](references/old-vs-new.md) | Legacy IOTA vs IOTA Rebased comparison table |
| [networks.md](references/networks.md) | Mainnet, Testnet, Devnet, Localnet — endpoints, faucets, CLI setup |
| [wallets.md](references/wallets.md) | IOTA Wallet (Chrome), Ledger, CLI wallet, migration from Firefly |
| [tokenomics.md](references/tokenomics.md) | Supply, fees, staking rewards |
| [staking.md](references/staking.md) | Staking, validators, DPoS participation |
| [migration.md](references/migration.md) | Migration from Stardust |

### Move Smart Contracts

| File | Content |
|------|---------|
| [move-basics.md](references/move-basics.md) | Move language intro with IOTA examples |
| [move-advanced.md](references/move-advanced.md) | Patterns: Witness, Hot Potato, Capability, OTW, Publisher, Generics |
| [move-objects-fields.md](references/move-objects-fields.md) | Object model, Dynamic fields, Tables, Bags, Transfer |
| [move-defi.md](references/move-defi.md) | Tokens (Coin), NFTs, DEX/AMM, Flash loans, Oracles |
| [move-testing.md](references/move-testing.md) | Unit tests, test_scenario, coverage, integration testing |
| [move-security.md](references/move-security.md) | Security checklist, access control, common mistakes |
| [move-gas.md](references/move-gas.md) | Gas optimization, owned vs shared objects, PTBs |
| [move-vs-sui.md](references/move-vs-sui.md) | IOTA Move vs Sui Move differences, migration guide |
| [smart-contracts.md](references/smart-contracts.md) | Create, test, deploy Move smart contracts |

### Tools & SDKs

| File | Content |
|------|---------|
| [sdks-tools.md](references/sdks-tools.md) | SDKs, CLI, GraphQL, IDE setup |
| [evm-layer2.md](references/evm-layer2.md) | IOTA EVM (Solidity on L2) |

### Ecosystem & Roadmap

| File | Content |
|------|---------|
| [ecosystem.md](references/ecosystem.md) | DeFi, identity, explorers, GitHub repos, community |
| [twin.md](references/twin.md) | TWIN — global digital trade infrastructure on IOTA |
| [roadmap-starfish.md](references/roadmap-starfish.md) | Starfish consensus, 2026 roadmap, protocol upgrades |
| [identity.md](references/identity.md) | Decentralized Identity (DID, Verifiable Credentials) on IOTA |

## Official Documentation

- **Docs**: https://docs.iota.org/developer/
- **GitHub**: https://github.com/iotaledger/iota
- **Cheat Sheet**: https://docs.iota.org/developer/dev-cheat-sheet
