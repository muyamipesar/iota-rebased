# IOTA Rebased — The Definitive AI-Ready Developer Guide

> **The most comprehensive, up-to-date resource for building on IOTA Rebased.** No outdated Tangle/Stardust confusion. Just clean, accurate, actionable documentation — ready for any AI coding assistant.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

## 🚀 What Is This?

IOTA underwent a **massive architectural transformation** in 2025 — from the legacy Tangle/Stardust protocol to **IOTA Rebased**: a Move VM-powered, object-oriented blockchain with DPoS consensus.

The problem? **Most IOTA documentation online is outdated.** Search results mix legacy concepts (Tangle, feeless, Coordinator, UTXO) with the new architecture (Move, Objects, Mysticeti, validators). This confuses developers and AI assistants alike.

**This repository solves that.** It provides:

- ✅ **Only IOTA Rebased** — zero legacy confusion
- ✅ **23 reference documents** covering everything from Move basics to DeFi patterns
- ✅ **Real code examples** from the official [iotaledger](https://github.com/iotaledger) repositories
- ✅ **AI-optimized format** — works with Claude, ChatGPT, Cursor, Windsurf, and any LLM
- ✅ **6,800+ lines** of curated, practical documentation

---

## 📚 What's Covered

### Core Architecture
| Document | Description |
|----------|-------------|
| [architecture.md](references/architecture.md) | Move VM, Object Model, Mysticeti/Starfish consensus, DPoS, PTBs, transaction lifecycle |
| [old-vs-new.md](references/old-vs-new.md) | **Essential** — Legacy IOTA vs Rebased comparison table, API/SDK migration maps |
| [tokenomics.md](references/tokenomics.md) | Supply dynamics, staking rewards, gas fees, storage deposits, governance |
| [networks.md](references/networks.md) | Mainnet, Testnet, Devnet endpoints, RPC URLs, faucets, Chain IDs |

### Move Language (7 documents)
| Document | Description |
|----------|-------------|
| [move-basics.md](references/move-basics.md) | Move 2024 syntax, packages, objects, ownership, collections |
| [move-advanced.md](references/move-advanced.md) | Witness, Hot Potato, Capability, Publisher, OTW, Generics |
| [move-objects-fields.md](references/move-objects-fields.md) | Object model deep dive, dynamic fields, wrapping, transfer |
| [move-defi.md](references/move-defi.md) | Tokens (Coin\<T\>), NFTs, DEX/AMM, flash loans, Kiosk, oracles |
| [move-testing.md](references/move-testing.md) | Unit tests, test_scenario, coverage, integration testing |
| [move-security.md](references/move-security.md) | Access control, validation, why Move prevents reentrancy |
| [move-gas.md](references/move-gas.md) | Gas mechanics, optimization, owned vs shared objects |

### Development Tools
| Document | Description |
|----------|-------------|
| [smart-contracts.md](references/smart-contracts.md) | Create → build → test → deploy workflow, 24 official examples |
| [sdks-tools.md](references/sdks-tools.md) | CLI, TypeScript SDK, Rust SDK, GraphQL API, IDE setup |
| [evm-layer2.md](references/evm-layer2.md) | IOTA EVM (Solidity L2), Chain IDs, Hardhat deploy, Move vs EVM |
| [move-vs-sui.md](references/move-vs-sui.md) | IOTA Move vs Sui Move differences, migration guide |

### Ecosystem & Infrastructure
| Document | Description |
|----------|-------------|
| [wallets.md](references/wallets.md) | IOTA Wallet (Chrome extension), Ledger, Firefly migration |
| [staking.md](references/staking.md) | DPoS staking guide (wallet + CLI), validator selection, rewards |
| [migration.md](references/migration.md) | Stardust → Rebased migration, NFT/Alias claiming, SDK mapping |
| [identity.md](references/identity.md) | Decentralized Identity (DID), Verifiable Credentials (W3C) |
| [twin.md](references/twin.md) | TWIN — global digital trade infrastructure on IOTA |
| [roadmap-starfish.md](references/roadmap-starfish.md) | Starfish consensus (next-gen), 2026 roadmap |
| [ecosystem.md](references/ecosystem.md) | Official repos, explorers, DPP, community resources |

---

## 🤖 Use With Any AI Assistant

### Claude Code / OpenClaw (Skill)

```bash
# Copy the skill folder to your workspace
cp -r references/ SKILL.md /path/to/your/workspace/skills/iota-rebased/
```

The AI will automatically use it when you ask about IOTA development.

### Claude Projects

Upload all `.md` files from `references/` as **Project Knowledge** files.

### OpenAI Custom GPTs

Upload all `.md` files from `references/` as **Knowledge** files in your GPT configuration.

### Cursor / Windsurf

```bash
# Copy references to your project
cp -r references/ /path/to/your/project/docs/iota/
```

The AI will reference them when you're coding.

### Any LLM (ChatGPT, Gemini, etc.)

Copy-paste the relevant `.md` file content into your conversation as context.

---

## ⚠️ Legacy vs Rebased — Know the Difference

If you see any of these terms, you're looking at **outdated** IOTA documentation:

| ❌ Legacy Term | ✅ Rebased Equivalent |
|---------------|----------------------|
| Tangle | IOTA blockchain |
| Coordinator | Mysticeti consensus (DPoS) |
| UTXO | Object Model |
| Feeless | Low fees (~0.005 IOTA per tx) |
| Hornet / Bee | IOTA node (Rust) |
| Firefly | IOTA Wallet (browser extension) |
| MIOTA | IOTA (1 IOTA = 1B Nanos) |
| Mana | Not used in Rebased |
| IOTA 2.0 | IOTA Rebased |

See [old-vs-new.md](references/old-vs-new.md) for the complete comparison.

---

## 📖 Official Resources

- **Documentation**: [docs.iota.org](https://docs.iota.org/)
- **GitHub**: [github.com/iotaledger](https://github.com/iotaledger)
- **Website**: [iota.org](https://www.iota.org/)
- **Explorer**: [explorer.iota.org](https://explorer.iota.org/)
- **TWIN**: [twin.org](https://www.twin.org/) | [twindev.org](https://twindev.org/)
- **Starfish Paper**: [eprint.iacr.org/2025/567](https://eprint.iacr.org/2025/567)
- **Whitepaper**: [IOTA Technical & Tokenomics](https://www.iota.org/pdf/IOTA_Technical_and_Tokenomics_Whitepaper.pdf)

### Community
- [Discord](https://discord.iota.org/) | [Builders Discord](https://builders-discord.iota.org/)
- [Telegram](https://t.me/IOTA_Official_Community)
- [Reddit](https://www.reddit.com/r/Iota/)
- [X (Twitter)](https://x.com/iota)

---

## 🤝 Contributing

Found outdated information? Want to add more examples? PRs are welcome!

Please ensure all contributions:
1. Reference **IOTA Rebased only** (not legacy Tangle/Stardust)
2. Use `iota::` namespaces in code examples (not `sui::`)
3. Include source links to [docs.iota.org](https://docs.iota.org/) or official repos
4. Are written in English

---

## 📄 License

MIT — Use freely, share widely. See [LICENSE](LICENSE).

---

*Built with ❤️ for the IOTA developer community. Last updated: February 2026.*
