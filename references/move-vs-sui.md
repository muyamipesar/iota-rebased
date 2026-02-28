# IOTA Move vs Sui Move: Key Differences

## Table of Contents

- [Overview](#overview)
- [1. Origin and Relationship](#1-origin-and-relationship)
- [2. Framework Namespace](#2-framework-namespace)
- [3. Native Token](#3-native-token)
- [4. Consensus](#4-consensus)
- [5. Gas and Fees](#5-gas-and-fees)
- [6. Validator Selection](#6-validator-selection)
- [7. Object Model and Move Language](#7-object-model-and-move-language)
- [8. Module and API Naming](#8-module-and-api-naming)
- [9. SDK and Tooling](#9-sdk-and-tooling)
- [10. Migration Guide: Sui → IOTA](#10-migration-guide-sui--iota)
- [11. Quick Reference Table](#11-quick-reference-table)
- [References](#references)

## Overview

IOTA Rebased is forked from Sui's codebase. The Move language and object model are fundamentally the same. The differences are in the **framework namespaces**, **native token**, **consensus improvements**, **gas pricing**, and **validator economics**. If you know Sui Move, you already know 95% of IOTA Move.

## 1. Origin and Relationship

- IOTA Rebased forked from Sui in 2024, adopting its Move VM, object model, and consensus
- The Move language version is identical: Move 2024 edition
- IOTA added improvements: resilient consensus (Starfish), fairer gas pricing, dynamic validator selection
- Code written for Sui can be ported to IOTA with mostly mechanical changes (namespace renaming)

## 2. Framework Namespace

The most visible difference. Every `sui::` becomes `iota::`:

| Sui | IOTA |
|-----|------|
| `sui::object` | `iota::object` |
| `sui::transfer` | `iota::transfer` |
| `sui::coin` | `iota::coin` |
| `sui::tx_context` | `iota::tx_context` |
| `sui::balance` | `iota::balance` |
| `sui::event` | `iota::event` |
| `sui::dynamic_field` | `iota::dynamic_field` |
| `sui::dynamic_object_field` | `iota::dynamic_object_field` |
| `sui::table` | `iota::table` |
| `sui::bag` | `iota::bag` |
| `sui::package` | `iota::package` |
| `sui::display` | `iota::display` |
| `sui::clock` | `iota::clock` |
| `sui::test_scenario` | `iota::test_scenario` |
| `sui::test_utils` | `iota::test_utils` |

### Move.toml Dependency

**Sui:**
```toml
[dependencies]
Sui = { git = "https://github.com/MystenLabs/sui.git", subdir = "crates/sui-framework/packages/sui-framework", rev = "..." }
```

**IOTA:**
```toml
[dependencies]
Iota = { git = "https://github.com/iotaledger/iota.git", subdir = "crates/iota-framework/packages/iota-framework", rev = "mainnet" }
```

### Auto-imports (Move 2024)

**Sui** auto-imports:
```move
use sui::object::{Self, ID, UID};
use sui::transfer;
use sui::tx_context::{Self, TxContext};
```

**IOTA** auto-imports:
```move
use iota::object::{Self, ID, UID};
use iota::transfer;
use iota::tx_context::{Self, TxContext};
```

## 3. Native Token

| | Sui | IOTA |
|---|-----|------|
| Token | SUI | IOTA |
| Module | `sui::sui::SUI` | `iota::iota::IOTA` |
| Smallest unit | MIST (1 SUI = 10⁹ MIST) | NANOS (1 IOTA = 10⁹ NANOS) |
| Total supply | 10 billion SUI | ~4.6 billion IOTA (migrated from legacy) |

```move
// Sui
use sui::sui::SUI;
use sui::coin::Coin;
fun pay(c: Coin<SUI>) { ... }

// IOTA
use iota::iota::IOTA;
use iota::coin::Coin;
fun pay(c: Coin<IOTA>) { ... }
```

## 4. Consensus

| | Sui | IOTA (current) | IOTA (planned) |
|---|-----|----------------|----------------|
| Consensus | Mysticeti | Mysticeti | Starfish |
| Architecture | DAG-based BFT | Same, with improvements | Encoded Cordial Dissemination |
| TPS (claimed) | ~100k+ | ~100k+ | ~150k+ |

Both currently use Mysticeti (DAG-based BFT consensus). IOTA is developing Starfish as a next-gen replacement with better fault tolerance and linear communication complexity. See [roadmap-starfish.md](roadmap-starfish.md).

## 5. Gas and Fees

| | Sui | IOTA |
|---|-----|------|
| Gas unit | MIST | NANOS |
| Fee destination | Staking rewards + burned | **100% burned** |
| Fee model | Computation + storage | Same |
| Typical simple tx | ~0.003 SUI | ~0.005 IOTA |
| Sponsored tx | Yes | Yes |
| Computation buckets | Yes | Yes |

IOTA burns 100% of gas fees, making the token deflationary under usage pressure.

## 6. Validator Selection

| | Sui | IOTA |
|---|-----|------|
| Validator model | DPoS | DPoS with dynamic selection |
| Validator selection | Stake-weighted | Dynamic committee selection (fairer) |
| Staking rewards source | Gas fees + inflation | Minted IOTA (~767k per epoch) |

IOTA's validator selection aims to prevent stake centralization by dynamically selecting committee members.

## 7. Object Model and Move Language

**These are identical:**
- Object ownership (owned, shared, immutable, wrapped)
- Dynamic fields and dynamic object fields
- Programmable Transaction Blocks (PTBs)
- Move 2024 edition (method syntax, enums, macros, index syntax)
- Abilities: `key`, `store`, `copy`, `drop`
- OTW (One-Time Witness) pattern
- All design patterns (Capability, Witness, Hot Potato, etc.)
- Maximum object size: 250 KB
- `Display` standard for NFTs

**No code changes needed** for Move logic — only namespace changes.

## 8. Module and API Naming

Some framework modules have IOTA-specific names:

| Sui | IOTA | Notes |
|-----|------|-------|
| `sui_system` | `iota_system` | System module for staking/validators |
| `sui::sui::SUI` | `iota::iota::IOTA` | Native coin type |
| `SuiSystemState` | `IotaSystemState` | System state object |

The framework source code lives at:
- **Sui:** `crates/sui-framework/packages/sui-framework/sources/`
- **IOTA:** `crates/iota-framework/packages/iota-framework/sources/`

You can browse IOTA's framework at: https://github.com/iotaledger/iota/tree/develop/crates/iota-framework

## 9. SDK and Tooling

| Tool | Sui | IOTA |
|------|-----|------|
| CLI | `sui` | `iota` |
| TypeScript SDK | `@mysten/sui` | `@iota/iota-sdk` |
| Rust SDK | `sui-sdk` | `iota-sdk` |
| Explorer | explorer.sui.io | explorer.iota.org |
| Faucet (testnet) | faucet.sui.io | faucet.testnet.iota.cafe |
| RPC | api.mainnet.sui.io | api.mainnet.iota.cafe |
| GraphQL | graphql.mainnet.sui.io | graphql.mainnet.iota.cafe |
| dApp scaffold | `pnpm create @mysten/dapp` | `pnpm create @iota/dapp` |
| Wallet | Sui Wallet | IOTA Wallet |

### CLI Commands

```bash
# Sui                              # IOTA
sui move new pkg                   iota move new pkg
sui move build                     iota move build
sui move test                      iota move test
sui client publish                 iota client publish
sui client call ...                iota client call ...
sui client active-address          iota client active-address
```

## 10. Migration Guide: Sui → IOTA

### Step-by-Step

1. **Find-and-replace namespaces:**
   ```bash
   # In your Move sources:
   sed -i 's/sui::/iota::/g' sources/*.move
   sed -i 's/use sui/use iota/g' sources/*.move
   ```

2. **Update Move.toml:**
   ```toml
   [dependencies]
   Iota = { git = "https://github.com/iotaledger/iota.git", subdir = "crates/iota-framework/packages/iota-framework", rev = "mainnet" }
   ```

3. **Replace native token type:**
   ```bash
   sed -i 's/sui::sui::SUI/iota::iota::IOTA/g' sources/*.move
   sed -i 's/Coin<SUI>/Coin<IOTA>/g' sources/*.move
   ```

4. **Update system module references:**
   ```bash
   sed -i 's/sui_system/iota_system/g' sources/*.move
   ```

5. **Build and test:**
   ```bash
   iota move build
   iota move test
   ```

6. **Update frontend SDK imports:**
   ```typescript
   // Before
   import { SuiClient } from '@mysten/sui/client';
   // After
   import { IotaClient } from '@iota/iota-sdk/client';
   ```

### What Might Break

- Custom types referencing `SUI` token — rename to `IOTA`
- Hardcoded Sui RPC endpoints — update to IOTA endpoints
- System state queries using Sui-specific types
- If you depend on Sui-specific framework modules not present in IOTA

### Use the Move Diff Tool

The community maintains a **Move Diff Tool** that shows side-by-side differences between Sui and IOTA framework source code:

🔗 [move-diff.vercel.app](https://move-diff.vercel.app/) — Select "Sui vs. IOTA" to explore differences

## 11. Quick Reference Table

| Aspect | Sui | IOTA Rebased |
|--------|-----|-------------|
| Framework prefix | `sui::` | `iota::` |
| Native coin | `SUI` | `IOTA` |
| Smallest unit | MIST | NANOS |
| CLI tool | `sui` | `iota` |
| Move edition | 2024 | 2024 |
| Object model | Identical | Identical |
| Consensus | Mysticeti | Mysticeti → Starfish |
| Gas fees burned | Partial | 100% |
| TypeScript SDK | `@mysten/sui` | `@iota/iota-sdk` |
| Mainnet RPC | api.mainnet.sui.io | api.mainnet.iota.cafe |
| Explorer | explorer.sui.io | explorer.iota.org |
| Diff tool | [move-diff.vercel.app](https://move-diff.vercel.app/) | Same |

## References

- [Move Diff Tool (Sui vs IOTA)](https://move-diff.vercel.app/)
- [IOTA Framework source](https://github.com/iotaledger/iota/tree/develop/crates/iota-framework)
- [IOTA Rebased announcement](https://blog.iota.org/iota-rebased-fast-forward/)
- [IOTA TypeScript SDK](https://docs.iota.org/developer/ts-sdk/)
- [← Prev: Gas Optimization](move-gas.md)
