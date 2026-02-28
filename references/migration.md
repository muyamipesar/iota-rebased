# Migration from Stardust to IOTA Rebased

## Table of Contents

- [Overview](#overview)
- [How the Migration Was Done](#how-the-migration-was-done)
- [What Migrated Automatically](#what-migrated-automatically)
- [What Needs Manual Claiming](#what-needs-manual-claiming)
- [Migrating from Firefly/Bloom to IOTA Wallet](#migrating-from-fireflybloom-to-iota-wallet)
- [Address Format Change](#address-format-change)
- [What Changed for Developers](#what-changed-for-developers)
- [SDK and API Migration Mapping](#sdk-and-api-migration-mapping)
- [NFTs, Alias Outputs, and Foundry Tokens](#nfts-alias-outputs-and-foundry-tokens)
- [What Happens If You Don't Migrate](#what-happens-if-you-dont-migrate)
- [Timeline](#timeline)
- [Edge Cases and Common Problems](#edge-cases-and-common-problems)
- [Developer Migration Checklist](#developer-migration-checklist)
- [References](#references)

## Overview

IOTA Rebased launched with an **automatic migration** of all Stardust UTXO assets into Move objects via a global snapshot. This documentation is primarily for **exchanges and dApp developers** — regular wallet users have migration handled by the wallet UI.

## How the Migration Was Done

The migration was performed via a deterministic snapshot-and-convert process:

1. **IOTA Foundation announced** milestone indices for snapshots
2. **Two global snapshots** taken of the IOTA Stardust network (via Hornet node software)
3. **IOTA EVM (L2)** paused deposits/withdrawals during the process
4. **Migration tool** converted all UTXOs into Move objects
5. **Move genesis** prepared with all converted objects from both snapshots
6. **Genesis blob published** and IOTA Rebased network started with it
7. **IOTA EVM reconnected** to the new L1; deposits/withdrawals re-enabled

All this happened automatically at the protocol level — no user action required for basic token access.

## What Migrated Automatically

| Asset Type | Status | Access |
|-----------|--------|--------|
| **IOTA tokens** | ✅ Available immediately | As `Coin<IOTA>` (`0x2::iota::IOTA`) |
| **Native tokens** (Stardust) | ✅ Available immediately | As Move objects |
| **Basic outputs** | ✅ Converted to Move objects | Directly accessible |
| **Ed25519 keys/addresses** | ✅ Same keys work | Just different encoding (hex) |

> **Decimal change**: IOTA token decimals changed from 6 to 9. All balances were multiplied by 1,000 during migration.

## What Needs Manual Claiming

Complex Stardust asset types were converted to Move objects but require **manual claiming** to unlock:

| Asset Type | Claiming Required | How to Claim |
|-----------|-------------------|-------------|
| **NFTs** | ✅ Yes | Via IOTA Wallet dashboard or SDK |
| **Alias outputs** | ✅ Yes | Via IOTA Wallet dashboard or SDK |
| **Foundry tokens** | ✅ Yes | Via SDK programmatically |
| **Time-locked tokens** | ✅ Yes (when unlock time arrives) | Via IOTA Wallet dashboard |
| **Storage deposit returns** | ✅ Yes | Part of claiming process |

### Claiming Process

**For wallet users:**
1. Import your mnemonic/Ledger into the new IOTA Wallet
2. Open the wallet dashboard
3. Follow the guided claiming flow for each complex asset

**For developers:**
Use the SDK with the [Stardust Move framework](https://docs.iota.org/developer/references/framework/stardust/address_unlock_condition) to claim programmatically. See [Claiming Stardust Assets](https://docs.iota.org/developer/stardust/claiming) for code examples.

## Migrating from Firefly/Bloom to IOTA Wallet

> ⚠️ **Firefly for IOTA is deprecated** and no longer supports the IOTA Rebased mainnet.

### Using a Mnemonic (24 words)

1. Install the [IOTA Wallet](https://chrome.google.com/webstore/detail/iota-wallet/lfhohhmmcplfddhojblhpbblpbcoecha) browser extension
2. Create a new profile → select "Import Existing Wallet"
3. Enter your 24-word mnemonic from Firefly/Bloom
4. Your tokens and assets will appear automatically
5. Complex assets may need claiming via the dashboard

### Using a Ledger Device

| Device | Min IOTA App Version | Notes |
|--------|---------------------|-------|
| Ledger Nano S | N/A | ❌ Not supported (insufficient memory) |
| Ledger Nano S Plus | v1.0.0+ | ✅ Supported |
| Ledger Nano X | v1.0.0+ | ✅ Supported |
| Ledger Stax | v1.0.0+ | ✅ Supported |

Steps:
1. Update Ledger firmware and IOTA app via Ledger Live
2. Install the IOTA Wallet browser extension
3. Connect Ledger → select "Connect Hardware Wallet"
4. Your accounts will be derived from the same keys

### If the In-App Updater Is Broken (Windows)

Download the latest Firefly version manually from [firefly.iota.org](https://firefly.iota.org/), export your mnemonic, then import into the new IOTA Wallet.

## Address Format Change

| Property | Old (Stardust) | New (Rebased) |
|----------|---------------|---------------|
| **Format** | Bech32 (`iota1qp...`) | Hex with 0x prefix (`0x7a3b...`) |
| **Scheme** | Ed25519 (BIP-32/44/39) | Ed25519 (same scheme) |
| **Key pairs** | Same | Same (no re-keying needed) |
| **Derivation** | Same | Same |

The underlying cryptographic keys are identical — only the encoding/representation changed. See [Addresses and Keys](https://docs.iota.org/developer/stardust/addresses) for conversion details.

### Supported Signature Schemes

IOTA Rebased supports:
- **Ed25519** (default, same as Stardust)
- **ECDSA Secp256k1**
- **ECDSA Secp256r1**
- **Multisig** (native, combining any of the above)

## What Changed for Developers

### ⚠️ All Integrations Are Broken

IOTA Rebased is a completely new tech stack. Everything must be rewritten:

| Component | Old (Stardust) | New (Rebased) |
|-----------|---------------|---------------|
| **Ledger model** | UTXO | Object-based (Move) |
| **Smart contracts** | None on L1 (ISC on L2) | Move VM on L1 |
| **API** | Chrysalis/Stardust REST | JSON-RPC / GraphQL / WebSocket |
| **SDK** | `iota.js`, `wallet.rs` | `@iota/iota-sdk` (TS), `iota-sdk` (Rust) |
| **Transaction model** | Consume UTXOs → Create UTXOs | Programmable Transaction Blocks on objects |
| **Address format** | Bech32 | Hex (0x prefix) |
| **Token decimals** | 6 | 9 |

### What Stays the Same

- **Ed25519 keys** — same key pairs work
- **Address derivation** — same BIP-32/44 scheme
- **Storage deposit concept** — similar mechanism preserved

## SDK and API Migration Mapping

| Old SDK/API | New Equivalent |
|------------|---------------|
| `iota.js` | `@iota/iota-sdk` (TypeScript) |
| `wallet.rs` | `iota-sdk` (Rust crate) |
| `iota.rs` (client lib) | `iota-sdk` Rust crate |
| REST API (`/api/v2/...`) | JSON-RPC (`iota_*` methods) |
| Event API | WebSocket subscriptions or GraphQL |
| UTXO balance query | `iota_getBalance`, `iota_getCoins` |
| Send transaction | `iota_executeTransactionBlock` |
| Get outputs | `iota_getOwnedObjects` |

### GraphQL Endpoints

Replace `api.` with `graphql.` in RPC URLs:
- Mainnet: `https://graphql.mainnet.iota.cafe`
- Testnet: `https://graphql.testnet.iota.cafe`

## NFTs, Alias Outputs, and Foundry Tokens

### NFTs

Stardust NFTs are converted into Move objects emulating the original NFT output structure. To claim:
1. The NFT exists as a Move object at your address
2. Call the claiming function via the Stardust Move framework
3. After claiming, the NFT becomes a standard IOTA object you can use in transactions

### Alias Outputs

Alias outputs (used for DID, governance, state channels) become Move objects with the original Alias structure. After claiming, you get:
- The alias object with its metadata
- Any tokens stored in the alias output
- Control via the same state controller / governor keys

### Foundry Tokens (Native Tokens)

Foundry outputs that managed custom native tokens are converted to Move objects. The token supply and metadata are preserved. Foundry controllers can claim and manage them via the Stardust Move framework.

## What Happens If You Don't Migrate

**Your tokens are NOT lost.** The migration was automatic:

- Basic IOTA tokens are accessible immediately with your existing keys
- Complex assets (NFTs, Aliases) remain as Move objects — **they wait for you indefinitely**
- There is **no deadline** to claim complex assets
- You just need to import your keys into a compatible wallet when ready
- The old Stardust network is shut down — you can't access it anymore, but your assets exist on Rebased

## Timeline

| Event | Status |
|-------|--------|
| Governance vote (December 2024) | ✅ Accepted |
| Public testnet launch | ✅ Live |
| Global snapshot of Stardust network | ✅ Completed |
| IOTA EVM pause for migration | ✅ Completed |
| IOTA Rebased mainnet launch | ✅ Live |
| IOTA EVM reconnected to Rebased L1 | ✅ Active |
| Complex asset claiming | 🔄 Ongoing (no deadline) |

## Edge Cases and Common Problems

### Problem: "I can't see my tokens in the new wallet"

**Solution:** Make sure you're using the correct mnemonic. The new wallet derives addresses using the same scheme — if the mnemonic is correct, your tokens should appear. Try checking the explorer with your address.

### Problem: "My NFT/Alias doesn't appear"

**Solution:** Complex assets need claiming. Open the wallet dashboard and follow the claiming flow. If using CLI/SDK, see the [Claiming guide](https://docs.iota.org/developer/stardust/claiming).

### Problem: "My Ledger Nano S isn't supported"

**Solution:** The Nano S has insufficient memory for the IOTA Rebased app. You need to upgrade to Nano S Plus, X, or Stax. Your keys can be recovered using the same 24-word mnemonic on a new device.

### Problem: "Address format looks different"

**Solution:** This is expected. The same key now shows as `0x...` instead of `iota1...`. Both encode the same underlying public key. See the [Addresses guide](https://docs.iota.org/developer/stardust/addresses).

### Problem: "Balance is 1000x what I expected"

**Solution:** IOTA decimals changed from 6 to 9. Your balance was multiplied by 1,000 to maintain the same value. If you had 100 IOTA before, you now have 100,000 IOTA — but the value is the same.

### Problem: "My time-locked tokens can't be claimed yet"

**Solution:** Time-locked tokens follow their original unlock schedule. You can delegate them to validators to earn staking rewards while waiting. They'll become claimable at the scheduled time.

## Developer Migration Checklist

- [ ] Install new IOTA CLI (`cargo install --locked --git https://github.com/iotaledger/iota.git --branch mainnet iota`)
- [ ] Install new SDKs (`@iota/iota-sdk` for TS, `iota-sdk` for Rust)
- [ ] Update all RPC endpoints to new format (JSON-RPC / GraphQL)
- [ ] Rewrite transaction logic from UTXO to Programmable Transaction Blocks
- [ ] Convert address handling from Bech32 to hex (0x prefix)
- [ ] Update balance queries to use `iota_getBalance` / `iota_getCoins`
- [ ] Adjust for new decimal precision (6 → 9, multiply by 1000)
- [ ] Claim any complex Stardust assets programmatically if needed
- [ ] Test thoroughly on testnet before mainnet deployment
- [ ] Update any address validation to accept hex format

## References

- [Stardust Migration](https://docs.iota.org/developer/stardust/stardust-migration)
- [Claiming Stardust Assets](https://docs.iota.org/developer/stardust/claiming)
- [Addresses and Keys](https://docs.iota.org/developer/stardust/addresses)
- [Stardust Framework Reference](https://docs.iota.org/developer/references/framework/stardust/address_unlock_condition)
- [FAQ: How to Migrate from Firefly](https://docs.iota.org/about-iota/FAQ)
- [IOTA Wallet Getting Started](https://docs.iota.org/users/iota-wallet/getting-started)
