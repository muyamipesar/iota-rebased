# IOTA Rebased Ecosystem

## Official Resources

| Resource | URL |
|----------|-----|
| **Website** | https://www.iota.org/ |
| **Documentation** | https://docs.iota.org/ |
| **Developer Hub** | https://docs.iota.org/developer/ |
| **GitHub** | https://github.com/iotaledger |

## Explorers

| Explorer | URL |
|----------|-----|
| **IOTA Explorer** (official) | https://explorer.iota.org/ |
| **IOTAScan** | https://iotascan.com/ |

## Wallets

| Wallet | Type | Notes |
|--------|------|-------|
| **IOTA Wallet** | Browser extension | Official wallet for IOTA Rebased |

## Faucets (Test Tokens)

| Network | Faucet URL |
|---------|-----------|
| **Testnet** | https://faucet.testnet.iota.cafe |
| **Devnet** | https://faucet.devnet.iota.cafe |
| **Localnet** | http://127.0.0.1:9123/gas |

## Networks & RPC Endpoints

| Network | JSON-RPC | GraphQL | WebSocket |
|---------|----------|---------|-----------|
| **Mainnet** | `https://api.mainnet.iota.cafe` | `https://graphql.mainnet.iota.cafe` | `wss://api.mainnet.iota.cafe` |
| **Testnet** | `https://api.testnet.iota.cafe` | `https://graphql.testnet.iota.cafe` | `wss://api.testnet.iota.cafe` |
| **Devnet** | `https://api.devnet.iota.cafe` | `https://graphql.devnet.iota.cafe` | `wss://api.devnet.iota.cafe` |

### Third-Party RPC Providers

- **Ankr**: `https://rpc.ankr.com/iota_mainnet` / `https://rpc.ankr.com/iota_testnet`
- **Monochain**: `https://rpc.mainnet.iota.monochain.p2p.org`
- **Nirvana Labs**: Hosted RPC nodes, archival services, indexers — https://nirvanalabs.io/

## Community

| Platform | Link |
|----------|------|
| **Builders Discord** | https://builders-discord.iota.org/ |
| **Community Discord** | https://discord.iota.org/ |
| **Telegram** | https://t.me/IOTA_Official_Community |
| **Reddit** | https://www.reddit.com/r/Iota/ |
| **X (Twitter)** | https://x.com/iota |
| **LinkedIn** | https://www.linkedin.com/company/iotafoundation/ |
| **YouTube** | https://www.youtube.com/c/iotafoundation |

## Key Ecosystem Projects

### TWIN — Trade Worldwide Information Network

The flagship real-world application on IOTA. Digital trade infrastructure connecting businesses, governments, and organizations across global supply chains. Live trade transactions anchored on IOTA Mainnet since January 2026.

- **Website:** [twin.org](https://www.twin.org/home)
- **Developer Portal:** [twindev.org](https://twindev.org/)
- **Whitepaper:** [TWIN Reference Architecture V1.0 (Jan 2026)](https://twindev.org/pdf/TWIN_Whitepaper_Reference_Architecture-V1.0-January-2026.pdf)
- **GitHub:** [github.com/twinfoundation](https://github.com/twinfoundation/docs)
- **TLIP (East Africa):** [tlip.io](https://www.tlip.io/)

→ Full details in [twin.md](twin.md)

### Digital Product Passport (DPP)

Traceable, verifiable product lifecycle records on IOTA — from manufacturing through repairs, resale, and recycling. Aligned with EU Digital Product Passport regulations (ESPR — Ecodesign for Sustainable Products Regulation).

- **Live Demo:** [dpp.demo.iota.org](https://dpp.demo.iota.org/introduction/1)
- **GitHub:** [github.com/iotaledger/dpp-demonstrator](https://github.com/iotaledger/dpp-demonstrator)
- Built on IOTA Trust Framework (identity, notarization, verifiable credentials)
- Enables: product origin tracking, repair history, recycling compliance, ESG reporting

### DeFi Ecosystem

| Project | Type | URL |
|---------|------|-----|
| **Swirl** | Liquid staking | [swirlstake.com](https://swirlstake.com/) |
| **Virtue** | CDP / Stablecoin | [virtue.money](https://virtue.money/) |
| **Pools** | DEX (spot trading) | [pools.finance](https://www.pools.finance/) |
| **CyberPerp** | Perpetual trading | [cyberperp.io](https://cyberperp.io/) |
| **Liquidlink** | Points/rewards dashboard | [iota.liquidlink.io](https://iota.liquidlink.io/dashboard) |

### Identity & Trust Products

| Product | Purpose | Docs |
|---------|---------|------|
| **IOTA Identity** | DIDs and Verifiable Credentials | [docs](https://docs.iota.org/developer/iota-identity/) |
| **IOTA Notarization** | Immutable document anchoring | [docs](https://docs.iota.org/developer/iota-notarization/) |
| **IOTA Hierarchies** | Trust delegation networks | [docs](https://docs.iota.org/developer/iota-hierarchies/) |
| **IOTA Gas Station** | Sponsored transactions | [docs](https://docs.iota.org/operator/gas-station/) |
| **IOTA Trust Framework** | Open-source toolkit for real-world apps | [docs](https://docs.iota.org/developer/iota-trust-framework) |

### Cross-Chain & Integrations

- **LayerZero + Stargate** — connects IOTA to 150+ blockchain networks (Ethereum, Solana, Base, BNB Chain)
- **BitGo** — institutional-grade custody for IOTA
- **Uphold** — buy/sell IOTA for US users
- **Turnkey** — enterprise wallet infrastructure

### Business Innovation Program

Selected participants building on IOTA Mainnet:
- **ObjectID** — verifiable product identities ([objectid.io](https://objectid.io/))
- **Orobo** — sustainability and traceability ([orobo.tech](https://orobo.tech/))
- **Impierce Technologies** — compliance infrastructure ([impierce.com](https://www.impierce.com/))

## Research & Academic

| Resource | Link |
|----------|------|
| **IOTA Technical & Tokenomics Whitepaper** | [PDF](https://www.iota.org/pdf/IOTA_Technical_and_Tokenomics_Whitepaper.pdf) |
| **Starfish Consensus Paper** | [IACR ePrint 2025/567](https://eprint.iacr.org/2025/567) |
| **Google Scholar** | https://scholar.google.com/citations?hl=en&user=_ZIH81gAAAAJ |
| **IOTA Trust Framework** | https://docs.iota.org/developer/iota-trust-framework |

## Developer Tools

| Tool | Description |
|------|-------------|
| **IOTA CLI** | Primary development tool (build, test, deploy, interact) |
| **TypeScript SDK** | `@iota/iota-sdk` — client, transactions, keypairs |
| **Rust SDK** | `iota-sdk` crate |
| **dApp Kit** | React framework: `pnpm create @iota/dapp` |
| **Move Analyzer** | VS Code extension for Move language support |
| **GraphQL API** | Query on-chain data with GraphQL |

## Official GitHub Repositories

### Core

| Repository | Language | Description |
|-----------|----------|-------------|
| [iota](https://github.com/iotaledger/iota) | Rust | Core node, Move framework, CLI, all protocol code |
| [starfish](https://github.com/iotaledger/starfish) | Rust | Starfish consensus — next-gen BFT protocol |
| [iota-rust-sdk](https://github.com/iotaledger/iota-rust-sdk) | Kotlin | Rust/Kotlin SDK for IOTA Rebased |
| [identity](https://github.com/iotaledger/identity) | Rust | DID and Verifiable Credentials (W3C) for IOTA MoveVM |
| [IIPs](https://github.com/iotaledger/IIPs) | Shell | IOTA Improvement Proposals |

### Tools & Services

| Repository | Language | Description |
|-----------|----------|-------------|
| [gas-station](https://github.com/iotaledger/gas-station) | Rust | Sponsored transactions service |
| [iota-names](https://github.com/iotaledger/iota-names) | TypeScript | Name service (like ENS for IOTA) |
| [iota-multisig-manager](https://github.com/iotaledger/iota-multisig-manager) | TypeScript | Multi-signature transaction manager |
| [notarization](https://github.com/iotaledger/notarization) | Rust | Document notarization on IOTA |
| [hierarchies](https://github.com/iotaledger/hierarchies) | Rust | Trust delegation hierarchies |
| [legacy-migration-tool](https://github.com/iotaledger/legacy-migration-tool) | TypeScript | Migrate from Stardust to Rebased |
| [ptb-builder](https://github.com/iotaledger/ptb-builder) | TypeScript | Visual PTB builder tool |
| [explorer](https://github.com/iotaledger/explorer) | TypeScript | IOTA block explorer |
| [dpp-demonstrator](https://github.com/iotaledger/dpp-demonstrator) | TypeScript | Digital Product Passport demo |
| [iota-supply-service](https://github.com/iotaledger/iota-supply-service) | Rust | REST service for IOTA circulating/total supply |

### Cryptography & Infrastructure

| Repository | Language | Description |
|-----------|----------|-------------|
| [stronghold.rs](https://github.com/iotaledger/stronghold.rs) | Rust | Secret management engine |
| [sd-jwt-payload](https://github.com/iotaledger/sd-jwt-payload) | Rust | Selective Disclosure JWT implementation |
| [crypto.rs](https://github.com/iotaledger/crypto.rs) | Rust | Cryptographic primitives for IOTA |
| [ledger-iota-app](https://github.com/iotaledger/ledger-iota-app) | C | Ledger hardware wallet app |
| [iota-caip](https://github.com/iotaledger/iota-caip) | Rust | CAIP (Chain Agnostic) standards for IOTA |

### Legacy (Still Active for Migration)

| Repository | Language | Description |
|-----------|----------|-------------|
| [firefly](https://github.com/iotaledger/firefly) | TypeScript | Legacy wallet (Stardust era) — use for key export only |
| [wasp](https://github.com/iotaledger/wasp) | Go | ISC/EVM L2 node |

## Learning Resources

| Resource | Link |
|----------|------|
| **Getting Started** | https://docs.iota.org/developer/getting-started/ |
| **Dev Cheat Sheet** | https://docs.iota.org/developer/dev-cheat-sheet |
| **Move CTF Challenges** | https://docs.iota.org/developer/iota-move-ctf/introduction |
| **From Solidity to Move** | https://docs.iota.org/developer/evm-to-move/ |
| **FAQ** | https://docs.iota.org/about-iota/FAQ |
| **IOTA Rebased Technical View** | https://blog.iota.org/iota-rebased-technical-view/ |
| **IOTA 2025 Review** | https://blog.iota.org/iota-2025-review/ |
