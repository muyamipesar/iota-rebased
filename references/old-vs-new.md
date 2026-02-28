# IOTA Legacy vs IOTA Rebased — Comparison

**This is the most important reference for avoiding confusion.** If you see old IOTA documentation, this table tells you what changed.

## Comparison Table

| Concept | IOTA Legacy (Tangle/Stardust) | IOTA Rebased | Note |
|---------|-------------------------------|--------------|------|
| **Architecture** | DAG (Tangle) | Blockchain (DAG-based mempool + Mysticeti consensus) | Completely different topology |
| **Ledger Model** | UTXO-based | Object-based (Move) | Objects replace UTXOs |
| **Consensus** | Coordinator (centralized) / planned IOTA 2.0 leaderless | Mysticeti BFT + DPoS (fully decentralized) | No more Coordinator dependency |
| **Smart Contracts (L1)** | ❌ None | ✅ Move VM (full L1 smart contracts) | Major new capability |
| **Smart Contracts (L2)** | IOTA Smart Contracts (ISC/EVM) | ISC/EVM continues on L2 | L2 EVM still available |
| **Transaction Fees** | Feeless | Small fees (~0.005 IOTA per tx), burned | Fees enable sustainable economics |
| **Token Supply** | Fixed (~4.6B MIOTA) | Dynamic: staking inflation (~6% yr1) offset by fee burning | Supply is no longer fixed |
| **Staking** | ❌ Not available | ✅ DPoS: delegate to validators, earn rewards | ~767k IOTA minted per epoch |
| **Validators** | Coordinator only | Up to 150 validators, 2M IOTA min stake | Fully decentralized |
| **Programming Language** | N/A (no L1 contracts) | Move (2024 edition) | Resource-oriented, statically verified |
| **Address Format** | Bech32 (`iota1...`) | Hex with `0x` prefix | Same keys, different encoding |
| **Parallel Execution** | Limited | ✅ Full parallel execution for owned objects | Major performance advantage |
| **Sponsored Transactions** | ❌ | ✅ Apps can pay fees for users | Great for onboarding |
| **Data Model** | Outputs (Basic, Alias, NFT, Foundry) | Move Objects (any shape/form) | Far more flexible |
| **Native Tokens** | Foundry outputs | Move Coin<T> standard | Simpler, more composable |
| **NFTs** | NFT Output type | Move objects with Display standard | More customizable |
| **Storage** | Storage deposit (locked tokens) | Storage deposit (redeemable, similar concept) | Mechanism preserved |
| **Node Software** | Hornet, Bee, Chronicle | IOTA node (Rust, based on Sui architecture) | Complete rewrite |
| **API** | REST + MQTT + Events | JSON-RPC + WebSocket + GraphQL | Modern API stack |
| **SDK** | iota.js, iota.rs, wallet.rs | @iota/iota-sdk (TS), iota-sdk (Rust) | New SDKs from scratch |
| **Wallet** | Firefly | IOTA Wallet (browser extension) | New wallet |
| **Explorer** | Explorer (Tangle-based) | [explorer.iota.org](https://explorer.iota.org/) | New explorer |

## Key Takeaways

1. **IOTA Rebased is NOT an upgrade of the old protocol** — it's a completely new architecture based on Sui's proven technology with IOTA-specific improvements.

2. **All old integrations are broken** — if you had code using iota.js, Hornet APIs, or UTXO-based logic, you must rewrite from scratch using the new SDKs.

3. **Same keys, new format** — your Ed25519 keys still work, but addresses are now `0x`-prefixed hex instead of Bech32.

4. **Feeless is gone** — but fees are extremely low (~0.005 IOTA) and staking rewards more than compensate.

5. **Move is the primary development language** — if you're building on IOTA L1, you write Move, not Solidity (Solidity is L2 only via ISC/EVM).

## Terms to Watch Out For

These terms belong to **legacy IOTA** and should NOT be used when discussing IOTA Rebased:

- ❌ Tangle → Use "IOTA network" or "IOTA blockchain"
- ❌ Coordinator/Coo → Replaced by Mysticeti consensus
- ❌ UTXO → Replaced by Object Model
- ❌ Feeless → Now has low fees
- ❌ Hornet/Bee → New node software
- ❌ Firefly → New IOTA Wallet
- ❌ Chrysalis/Stardust → Legacy protocol versions
- ❌ IOTA 2.0 → Replaced by IOTA Rebased
- ❌ Mana → Not used in Rebased
- ❌ MIOTA → Just IOTA (1 IOTA = 1,000,000,000 Nanos)

## API Migration Map

| Legacy API | Rebased Equivalent | Notes |
|------------|-------------------|-------|
| REST API (`/api/v1/...`) | JSON-RPC (`iota_getObject`, etc.) | Completely different protocol |
| MQTT events | WebSocket subscriptions | `wss://api.mainnet.iota.cafe` |
| Event API | GraphQL API | `https://graphql.mainnet.iota.cafe` |
| `/tips` | N/A | No tips in object model |
| `/outputs` | `iota_getOwnedObjects` | Objects replace outputs |
| `/milestones` | `iota_getCheckpoints` | Checkpoints replace milestones |
| `/transactions` | `iota_getTransactionBlock` | Completely different tx format |
| Node health | `iota_getLatestCheckpointSequenceNumber` | |

### Key API Differences

```
# Legacy: query an address's outputs
GET https://api.stardust.iota.org/api/v1/addresses/iota1.../outputs

# Rebased: query an address's objects
curl -X POST https://api.mainnet.iota.cafe -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"iota_getOwnedObjects","params":["0xADDRESS",null,null,null]}'
```

## SDK Migration Map

| Legacy SDK | Rebased SDK | Install |
|-----------|-------------|---------|
| `@iota/iota.js` | `@iota/iota-sdk` | `npm i @iota/iota-sdk` |
| `iota-client` (Rust) | `iota-sdk` (Rust crate) | `cargo add iota-sdk` |
| `wallet.rs` | `@iota/iota-sdk` (TS) or CLI | No separate wallet lib |
| `iota.rs` (bindings) | `@iota/iota-sdk` + `iota-rust-sdk` | |
| No Python SDK | `pyiota` (community) | `pip install pyiota` |

### Code Comparison

```typescript
// ❌ Legacy (iota.js) — no longer works
import { ClientBuilder } from '@iota/client';
const client = new ClientBuilder().node('https://api.stardust.iota.org').build();
const info = await client.getInfo();
const outputs = await client.basicOutputIds([{ address: 'iota1...' }]);

// ✅ Rebased (@iota/iota-sdk)
import { IotaClient } from '@iota/iota-sdk/client';
const client = new IotaClient({ url: 'https://api.mainnet.iota.cafe' });
const objects = await client.getOwnedObjects({ owner: '0x...' });
const balance = await client.getBalance({ owner: '0x...' });
```

## Wallet Migration

| Step | Details |
|------|---------|
| **1. Export keys** | Export your Ed25519 private key from Firefly (Settings → Security → Export Stronghold) |
| **2. Address format** | Your Bech32 `iota1...` address becomes `0x`-prefixed hex (same underlying key) |
| **3. Install new wallet** | [IOTA Wallet](https://chrome.google.com/webstore/detail/iota-wallet) browser extension |
| **4. Import key** | Import your Ed25519 key into the new wallet |
| **5. Migration claim** | If you held tokens pre-Rebased, claim them via the migration process (see [migration.md](migration.md)) |

### Token Unit Changes

| Legacy | Rebased |
|--------|---------|
| 1 MIOTA = 1,000,000 IOTA | 1 IOTA = 1,000,000,000 Nanos |
| Display: MIOTA, KIOTA, IOTA | Display: IOTA, Nanos |
| ~4.6 billion MIOTA total | ~4.6 billion IOTA base + staking inflation |

> **Important:** The denomination changed. What was "1 MIOTA" in legacy is approximately "1 IOTA" in Rebased. Check exact conversion ratios in the migration documentation.

## Data Model Migration

| Legacy Concept | Rebased Equivalent |
|---------------|-------------------|
| Basic Output | Owned Object (any struct with `key`) |
| Alias Output | Package / shared object with `AdminCap` |
| NFT Output | Object with `key + store` + `Display` |
| Foundry Output | `TreasuryCap<T>` + `CoinMetadata<T>` |
| Storage Deposit | Storage rebate (returned when object deleted) |
| Unlock Conditions | Move logic (time locks, capability checks) |
| Native Tokens (on output) | `Coin<T>` objects |
| Tag / Metadata (on output) | Struct fields + dynamic fields |

## Node Operator Migration

| Legacy | Rebased |
|--------|---------|
| Hornet (Go) | `iota` node (Rust) |
| INX plugins | GraphQL + custom indexers |
| Chronicle (permanode) | Archival node + GraphQL |
| Config: `config.json` | Config: YAML or TOML |
| Peering: manual | Peer discovery: automatic |
| Dashboard: `:8081` | No built-in dashboard (use explorer) |

```bash
# Install and run an IOTA Rebased node
cargo install --locked --git https://github.com/iotaledger/iota.git --branch mainnet iota
iota start --network mainnet
```
