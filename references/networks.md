# IOTA Networks Guide

## Table of Contents

- [Overview](#overview)
- [1. Network Summary Table](#1-network-summary-table)
- [2. Mainnet](#2-mainnet)
- [3. Testnet](#3-testnet)
- [4. Devnet](#4-devnet)
- [5. Local Network](#5-local-network)
- [6. Connecting via CLI](#6-connecting-via-cli)
- [7. Connecting via SDK](#7-connecting-via-sdk)
- [8. Third-Party RPC Providers](#8-third-party-rpc-providers)
- [9. Which Network Should I Use?](#9-which-network-should-i-use)
- [References](#references)

## Overview

IOTA Rebased provides four network environments. All use the same protocol (Move VM, object model, DPoS consensus). The only differences are data persistence, token value, and validator sets.

## 1. Network Summary Table

| Network | JSON RPC | WebSocket | GraphQL | Faucet | Explorer |
|---------|----------|-----------|---------|--------|----------|
| **Mainnet** | `https://api.mainnet.iota.cafe` | `wss://api.mainnet.iota.cafe` | `https://graphql.mainnet.iota.cafe` | — | [explorer.iota.org](https://explorer.iota.org/) |
| **Testnet** | `https://api.testnet.iota.cafe` | `wss://api.testnet.iota.cafe` | `https://graphql.testnet.iota.cafe` | [faucet.testnet.iota.cafe](https://faucet.testnet.iota.cafe) | [explorer (testnet)](https://explorer.iota.org/?network=testnet) |
| **Devnet** | `https://api.devnet.iota.cafe` | `wss://api.devnet.iota.cafe` | `https://graphql.devnet.iota.cafe` | [faucet.devnet.iota.cafe](https://faucet.devnet.iota.cafe) | [explorer (devnet)](https://explorer.rebased.iota.org/?network=devnet) |
| **Localnet** | `http://127.0.0.1:9000` | `ws://127.0.0.1:9000` | `http://127.0.0.1:8000` | `http://127.0.0.1:9123/gas` | [local explorer](https://explorer.rebased.iota.org/?network=http://127.0.0.1:9000) |

### Indexer Endpoints

| Network | Indexer URL |
|---------|------------|
| Mainnet | `https://indexer.mainnet.iota.cafe` |
| Testnet | `https://indexer.testnet.iota.cafe` |
| Devnet | `https://indexer.devnet.iota.cafe` |
| Localnet | `http://127.0.0.1:9124` |

## 2. Mainnet

The production network where tokens have real value.

- **Protocol:** Rebased (Move VM, DPoS, Mysticeti consensus)
- **Base token:** IOTA (real value)
- **Validators:** Decentralized set of up to 100 elected validators
- **Data persistence:** Permanent

```bash
# Connect CLI to Mainnet
iota client new-env --alias mainnet --rpc https://api.mainnet.iota.cafe
iota client switch --env mainnet
```

> **⚠️ Mainnet uses real tokens.** Always test on Testnet/Devnet first.

## 3. Testnet

Battle-tested releases move from Devnet to Testnet. Mirrors Mainnet conditions with a decentralized validator set.

- **Base token:** IOTA (no real value)
- **Validators:** Decentralized set (similar to Mainnet)
- **Data persistence:** Long-lived but may be wiped on major upgrades
- **Faucet:** Available for requesting test tokens

```bash
# Connect CLI to Testnet
iota client new-env --alias testnet --rpc https://api.testnet.iota.cafe
iota client switch --env testnet

# Request test tokens via CLI
iota client faucet

# Or via cURL
curl -X POST https://faucet.testnet.iota.cafe/v1/gas \
    -H "Content-Type: application/json" \
    -d '{"FixedAmountRequest":{"recipient":"0x<YOUR_ADDRESS>"}}'
```

### Testnet Faucet Web UI

Visit [faucet.testnet.iota.cafe](https://faucet.testnet.iota.cafe) in your browser, paste your address, and request tokens.

## 4. Devnet

The latest stable release. Receives new features first before they reach Testnet.

- **Base token:** IOTA (no real value)
- **Validators:** 4 nodes operated by IOTA Foundation
- **Data persistence:** **Not guaranteed** — wiped regularly with software updates
- **Use for:** Early testing, experimenting with latest features

```bash
# Connect CLI to Devnet
iota client new-env --alias devnet --rpc https://api.devnet.iota.cafe
iota client switch --env devnet

# Request test tokens
iota client faucet

# Or via cURL
curl -X POST https://faucet.devnet.iota.cafe/v1/gas \
    -H "Content-Type: application/json" \
    -d '{"FixedAmountRequest":{"recipient":"0x<YOUR_ADDRESS>"}}'
```

> **Warning:** Devnet data is wiped regularly. Don't rely on Devnet for persistent state.

## 5. Local Network

Run a local IOTA network for development. Full control, no rate limits, instant transactions.

### Starting a Local Network

```bash
# Start local network with faucet
RUST_LOG="off,iota_node=info" cargo run --bin iota start --force-regenesis --with-faucet

# Or if you have the binary installed:
iota start --force-regenesis --with-faucet
```

This starts:
- A validator node on `http://127.0.0.1:9000`
- A faucet on `http://127.0.0.1:9123`
- An indexer on `http://127.0.0.1:9124`
- A GraphQL endpoint on `http://127.0.0.1:8000`

### Connect CLI to Local Network

```bash
iota client new-env --alias localnet --rpc http://127.0.0.1:9000
iota client switch --env localnet
```

### Request Local Tokens

```bash
# Via CLI
iota client faucet

# Via cURL
curl -X POST http://127.0.0.1:9123/gas \
    -H "Content-Type: application/json" \
    -d '{"FixedAmountRequest":{"recipient":"0x<YOUR_ADDRESS>"}}'
```

### View in Explorer

Open [explorer.rebased.iota.org/?network=http://127.0.0.1:9000](https://explorer.rebased.iota.org/?network=http://127.0.0.1:9000) in your browser.

## 6. Connecting via CLI

### First-Time Setup

When you run `iota client` for the first time, it walks you through setup:

1. Select a default network (`mainnet`, `testnet`, `devnet`, `localnet`, or custom URL)
2. Choose a key scheme (Ed25519 recommended)
3. Receive your address and secret recovery phrase

### Managing Environments

```bash
# List all configured environments
iota client envs

# Add a new environment
iota client new-env --alias <NAME> --rpc <RPC_URL>

# Switch environments
iota client switch --env <NAME>

# Check current environment
iota client envs
# Shows (active) next to the current one
```

### Check Balance and Address

```bash
iota client active-address
iota client balance
iota client gas    # List gas coin objects
```

## 7. Connecting via SDK

### TypeScript

```typescript
import { getFullnodeUrl, IotaClient } from '@iota/iota-sdk/client';

// Use built-in URL helper
const client = new IotaClient({ url: getFullnodeUrl('testnet') });

// Or specify manually
const mainnetClient = new IotaClient({
    url: 'https://api.mainnet.iota.cafe'
});

// Query example
const balance = await client.getBalance({
    owner: '0x<ADDRESS>',
});
console.log(balance);
```

### Rust

```rust
use iota_sdk::IotaClientBuilder;

let client = IotaClientBuilder::default()
    .build("https://api.testnet.iota.cafe")
    .await?;

let balance = client
    .coin_read_api()
    .get_balance("0x<ADDRESS>".parse()?, None)
    .await?;
```

## 8. Third-Party RPC Providers

For production applications, consider using third-party RPC providers for better reliability and no rate limits:

| Provider | Mainnet URL | Testnet URL |
|----------|------------|-------------|
| **Ankr** | `https://rpc.ankr.com/iota_mainnet` | `https://rpc.ankr.com/iota_testnet` |
| **Monochain** | `https://rpc.mainnet.iota.monochain.p2p.org` | — |
| **Nirvana Labs** | Custom (hosted nodes, archival, indexers) | Custom |

> **Tip:** For production apps, run your own IOTA full node or use a premium RPC provider to avoid rate limits and ensure reliability.

## 9. Which Network Should I Use?

| Stage | Network | Why |
|-------|---------|-----|
| **Learning / Prototyping** | Devnet | Latest features, fast iteration, free tokens |
| **Testing before launch** | Testnet | Realistic conditions, decentralized validators |
| **Production** | Mainnet | Real value, permanent data |
| **Local development** | Localnet | Full control, instant, no rate limits |

### Recommended Developer Workflow

```
1. Develop & unit test locally (iota move test)
2. Deploy to Devnet for integration testing
3. Deploy to Testnet for pre-production testing
4. Deploy to Mainnet for production
```

## References

- [Network Overview](https://docs.iota.org/developer/network-overview)
- [Connect to a Network](https://docs.iota.org/developer/getting-started/connect)
- [Local Network Setup](https://docs.iota.org/developer/getting-started/local-network)
- [IOTA CLI reference](https://docs.iota.org/developer/references/cli/client)
