# IOTA EVM (Layer 2)

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Networks & Endpoints](#networks--endpoints)
- [When to Use Move (L1) vs EVM (L2)](#when-to-use-move-l1-vs-evm-l2)
- [Getting Started](#getting-started)
- [Deploy with Hardhat](#deploy-with-hardhat)
- [Bridging L1 ↔ L2](#bridging-l1--l2)
- [ISC Features](#isc-features)
- [Gas Costs: L1 vs L2](#gas-costs-l1-vs-l2)
- [Available Tooling on IOTA EVM](#available-tooling-on-iota-evm)
- [From Solidity/EVM to Move](#from-solidityevm-to-move)
- [References](#references)

## Overview

IOTA EVM is a **Layer 2** solution running on top of IOTA Rebased via the IOTA Smart Contracts (ISC) protocol. It provides full EVM/Solidity compatibility for developers who want to port existing Ethereum dApps or prefer Solidity.

ISC allows multiple L2 chains to run in parallel, each with its own state and smart contracts, enabling horizontal scaling of dApps.

## Architecture

- ISC chains run as L2 on top of the IOTA L1 (Rebased)
- Each ISC chain has its own state, validators, and smart contracts
- Multiple ISC chains can run in parallel (horizontal scaling)
- Chains communicate with L1 and with each other
- Validators process smart contract requests and write state changes into the chain
- L2 chains update their state collectively and interact with L1 and other L2 chains

```
┌─────────────────────────────────────────────┐
│              IOTA L1 (Rebased)              │
│         Move VM · Object Model · DPoS       │
└─────────┬───────────┬───────────┬───────────┘
          │           │           │
    ┌─────▼─────┐ ┌───▼─────┐ ┌──▼──────┐
    │ ISC Chain │ │ISC Chain│ │ISC Chain│
    │ (IOTA EVM)│ │  (...)  │ │  (...)  │
    │ Solidity  │ │         │ │         │
    └───────────┘ └─────────┘ └─────────┘
```

## Networks & Endpoints

### IOTA EVM Mainnet

| Property | Value |
|----------|-------|
| **Chain ID** | `8822` |
| **RPC URL** | `https://json-rpc.evm.iotaledger.net` |
| **Ankr RPC** | `https://rpc.ankr.com/iota_evm` |
| **Explorer** | [explorer.evm.iota.org](https://explorer.evm.iota.org) |
| **Bridge** | [evm-bridge.iota.org](https://evm-bridge.iota.org) |
| **Base Token** | IOTA |
| **Protocol** | ISC / EVM |

### IOTA EVM Testnet

| Property | Value |
|----------|-------|
| **Chain ID** | `1075` |
| **RPC URL** | `https://json-rpc.evm.testnet.iotaledger.net` |
| **Explorer** | [explorer.evm.testnet.iotaledger.net](https://explorer.evm.testnet.iotaledger.net) |
| **Base Token** | IOTA (test) |

> ⚠️ The testnet is subject to occasional resets (no data retention), usually announced with a one-week grace period.

### Adding to MetaMask

1. Open MetaMask → Networks → Add Network
2. Enter:
   - **Network Name**: IOTA EVM
   - **RPC URL**: `https://json-rpc.evm.iotaledger.net`
   - **Chain ID**: `8822`
   - **Currency Symbol**: IOTA
   - **Explorer URL**: `https://explorer.evm.iota.org`

## When to Use Move (L1) vs EVM (L2)

| Criteria | Move (L1) | EVM/Solidity (L2) |
|----------|-----------|-------------------|
| **New project from scratch** | ✅ Preferred | Possible but suboptimal |
| **Porting existing Solidity code** | Requires rewrite | ✅ Direct deployment |
| **Maximum performance** | ✅ Parallel execution, no L2 overhead | Sequential, L2 overhead |
| **Asset-oriented logic** | ✅ Resource types are native | Requires careful patterns |
| **DeFi primitives** | ✅ Native object model | Standard EVM patterns |
| **Ecosystem tooling (Hardhat, etc.)** | Move-specific tools | ✅ Full Ethereum tooling |
| **Developer familiarity** | Learning curve for Move | ✅ If you know Solidity |
| **Interop with Ethereum ecosystem** | Limited | ✅ EVM compatible |

**Recommendation**: Use Move for new projects. Use EVM for porting existing Solidity code or when you need Ethereum tooling compatibility.

## Getting Started

IOTA EVM supports standard EVM tooling:

- **Hardhat**, **Foundry**, **Remix** — all work as expected
- **MetaMask** — connect with IOTA EVM chain RPC
- **Solidity** — standard Solidity contracts deploy directly

### Tooling Considerations

- Use the correct JSON-RPC endpoint URL for your target network
- Use the correct Chain ID (8822 for mainnet, 1075 for testnet)
- Gas fees are handled at the ISC chain level, not the EVM level
- The chain rejects requests with a gas price different from the one specified by the chain

## Deploy with Hardhat

### 1. Setup Project

```bash
mkdir iota-evm-project && cd iota-evm-project
npm init -y
npm install --save-dev hardhat @nomicfoundation/hardhat-toolbox
npx hardhat init
```

### 2. Configure `hardhat.config.js`

```javascript
require("@nomicfoundation/hardhat-toolbox");

module.exports = {
  solidity: "0.8.24",
  networks: {
    iotaEvm: {
      url: "https://json-rpc.evm.iotaledger.net",
      chainId: 8822,
      accounts: [process.env.PRIVATE_KEY],
    },
    iotaEvmTestnet: {
      url: "https://json-rpc.evm.testnet.iotaledger.net",
      chainId: 1075,
      accounts: [process.env.PRIVATE_KEY],
    },
  },
};
```

### 3. Deploy

```bash
# Deploy to testnet
npx hardhat run scripts/deploy.js --network iotaEvmTestnet

# Deploy to mainnet
npx hardhat run scripts/deploy.js --network iotaEvm
```

### 4. Verify on Explorer

```bash
npx hardhat verify --network iotaEvm DEPLOYED_CONTRACT_ADDRESS
```

## Bridging L1 ↔ L2

### IOTA EVM Bridge

The [IOTA EVM Bridge](https://evm-bridge.iota.org) allows you to:

- **Deposit** IOTA tokens from L1 to IOTA EVM (L2)
- **Withdraw** IOTA tokens from IOTA EVM (L2) back to L1
- **Wrap** IOTA into wIOTA (ERC-20 compatible wrapper)

### L1 → L2 (On-Ledger Requests)

On-ledger requests are L1 transactions that ISC validator nodes retrieve from the ledger. This is the **only way** to transfer assets to a chain or between chains. The L1 acts as an arbiter guaranteeing transaction validity.

### L2 → L1 (Withdrawals)

Use the bridge UI or call the ISC core contract to withdraw assets back to L1.

### Off-Ledger Requests

If all necessary assets are already on the L2 chain, you can send requests directly to the chain's validator nodes, bypassing L1. This is **faster** (no L1 confirmation needed) and preferred when no new assets need to be deposited.

## ISC Features

- **Sandbox interface**: Smart contracts access chain state, native assets, and cross-chain communication
- **On-ledger requests**: L1 transactions picked up by ISC validators (for deposits & cross-chain)
- **Off-ledger requests**: Direct to validators (faster, for operations with on-chain assets)
- **Gas**: Computation measured in gas units with a gas budget per request
- **Entry points**: View-only or state-modifying, triggered by signed requests
- **Cross-chain calls**: Contracts on the same chain call each other synchronously; cross-chain is asynchronous

## Gas Costs: L1 vs L2

| Aspect | Move L1 | IOTA EVM (L2) |
|--------|---------|---------------|
| **Avg. transfer cost** | ~0.005 IOTA | ~0.01-0.05 IOTA (varies) |
| **Fee mechanism** | Burned (deflationary) | Paid to ISC chain |
| **Gas pricing** | Fixed reference price (1000 Nanos/unit) | Set by ISC chain config |
| **Storage** | Rebatable deposit | Standard EVM storage model |
| **Parallelism** | ✅ Owned objects parallel | ❌ Sequential EVM execution |
| **Sponsored txs** | ✅ Native support | ❌ Not natively supported |

> L1 Move transactions are generally cheaper and faster than L2 EVM transactions due to no L2 overhead.

## Available Tooling on IOTA EVM

### Oracles

| Provider | Contract |
|----------|----------|
| **Pyth** | [0x8D254a...](https://explorer.evm.iota.org/address/0x8D254a21b3C86D32F7179855531CE99164721933) |
| **Supra** (Pull) | [0x2FA6Db...](https://explorer.evm.iota.org/address/0x2FA6DbFe4291136Cf272E1A3294362b6651e8517) |

### Other Tools

- **Safe Wallet**: [safe.iotaledger.net](https://safe.iotaledger.net/) — MultiSig solution
- **Multicall3**: Aggregate multiple contract reads/writes in a single call
- **Subgraphs**: Available via [Goldsky](https://goldsky.com) for indexing
- **Ankr RPC**: Enterprise-grade globally distributed endpoints

## From Solidity/EVM to Move

If you're considering migrating from Solidity to Move on L1, see the official guide: [EVM to Move](https://docs.iota.org/developer/evm-to-move/).

Key mapping concepts:

| Solidity/EVM | Move/IOTA |
|-------------|-----------|
| `mapping(address => uint)` | Dynamic fields or `Table` |
| `msg.sender` | `tx_context::sender(ctx)` |
| `payable` | `Coin<IOTA>` parameter |
| `require()` | `assert!()` |
| ERC-20 | `iota::coin` (Coin standard) |
| ERC-721 | Objects with `key + store` |
| `contract` | `module` |

## References

- [IOTA EVM Introduction](https://docs.iota.org/developer/iota-evm/)
- [Compatible Tools](https://docs.iota.org/developer/iota-evm/getting-started/tools)
- [ISC White Paper](https://files.iota.org/papers/ISC_WP_Nov_10_2021.pdf)
- [From Solidity/EVM to Move](https://docs.iota.org/developer/evm-to-move/)
- [IOTA EVM Bridge](https://evm-bridge.iota.org)
