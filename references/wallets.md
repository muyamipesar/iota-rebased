# IOTA Wallets Guide

## Table of Contents

- [Overview](#overview)
- [1. IOTA Wallet (Chrome Extension)](#1-iota-wallet-chrome-extension)
- [2. Creating a New Wallet](#2-creating-a-new-wallet)
- [3. Importing an Existing Wallet](#3-importing-an-existing-wallet)
- [4. Sending and Receiving IOTA](#4-sending-and-receiving-iota)
- [5. Staking from the Wallet](#5-staking-from-the-wallet)
- [6. Connecting to dApps](#6-connecting-to-dapps)
- [7. Migration from Firefly](#7-migration-from-firefly)
- [8. Hardware Wallets](#8-hardware-wallets)
- [9. CLI Wallet](#9-cli-wallet)
- [10. Security Best Practices](#10-security-best-practices)
- [References](#references)

## Overview

IOTA Rebased uses a **new wallet** — the **IOTA Wallet** Chrome extension. The legacy Firefly wallet used with IOTA Stardust/Chrysalis is **no longer supported** for IOTA Rebased.

> **Important:** IOTA Rebased uses an entirely different protocol (Move VM, object model, DPoS) than legacy IOTA. Legacy wallets cannot interact with the Rebased network.

## 1. IOTA Wallet (Chrome Extension)

The official wallet for IOTA Rebased is a browser extension for Chromium-based browsers (Chrome, Brave, Edge).

**Features:**
- Create and manage multiple profiles and addresses
- Send/receive IOTA tokens and NFTs
- Stake IOTA and earn rewards
- Connect to dApps via the wallet standard
- Import private keys from other wallets
- View coins, tokens, and NFTs in one place

### Installation

1. Open a Chromium-based browser (Chrome, Brave, Edge)
2. Visit the [IOTA Wallet on Chrome Web Store](https://chromewebstore.google.com/detail/iota-wallet/iidjkmdceolghepehaaddojmnjnkkija)
3. Click **Add to Chrome**
4. Approve the permissions and confirm **Add Extension**
5. The IOTA Wallet icon appears in your browser toolbar

## 2. Creating a New Wallet

1. Click the IOTA Wallet icon → **Get Started**
2. Choose **Create a new wallet**
3. Select authentication method:
   - **Mnemonic** (recommended) — 12-word recovery phrase
   - **Passkey** — biometric/device-based authentication
4. Enter a password for the wallet
5. Accept the Terms of Service → **Create Wallet**
6. **Copy and securely save your recovery phrase (mnemonic)**
7. Confirm you saved the phrase → **Open Wallet**

> ⚠️ **Your mnemonic recovery phrase is the ONLY way to recover your wallet.** If lost, your assets are permanently inaccessible. Write it down on paper and store it securely. Never share it digitally.

## 3. Importing an Existing Wallet

### Import via Private Key

1. Open IOTA Wallet → Settings → **Import Private Key**
2. Enter your 32-byte or 64-byte private key
3. The wallet derives the corresponding address

### Import via Mnemonic

1. Open IOTA Wallet → **Import an Existing Wallet**
2. Enter your 12 or 24-word mnemonic phrase
3. Set a new password
4. The wallet restores all accounts derived from that mnemonic

### Key Schemes Supported

| Scheme | Description |
|--------|-------------|
| Ed25519 | Default, recommended |
| Secp256k1 | Bitcoin-compatible |
| Secp256r1 | WebAuthn/Passkey compatible |

## 4. Sending and Receiving IOTA

### Receive

1. Open the wallet → click **Receive**
2. Copy your address or show the QR code
3. Share the address with the sender

### Send

1. Open the wallet → click **Send**
2. Enter the recipient address
3. Enter the amount of IOTA to send
4. Review the transaction details and gas fee
5. Click **Send** and confirm with your password

### Send NFTs

1. Go to the **NFTs** tab in your wallet
2. Select the NFT you want to send
3. Click **Send** → enter recipient address → confirm

## 5. Staking from the Wallet

IOTA Rebased uses Delegated Proof of Stake (DPoS). You can stake IOTA to validators and earn rewards.

1. Open the wallet → go to the **Staking** section
2. Browse available validators
3. Select a validator → enter the amount to stake
4. Confirm the staking transaction

**Staking details:**
- Rewards are distributed per epoch (~24 hours)
- ~767,000 IOTA minted per epoch for staking rewards
- You can unstake at any time (effective at next epoch boundary)
- Staked IOTA remains in your custody — delegated, not locked

## 6. Connecting to dApps

The IOTA Wallet implements the wallet standard for dApp connectivity:

1. Visit a dApp that supports IOTA (e.g., a DEX, NFT marketplace)
2. Click **Connect Wallet** on the dApp
3. Select **IOTA Wallet** from the wallet options
4. Approve the connection in the wallet popup
5. When the dApp requests a transaction, review and approve in the popup

### For Developers

```typescript
import { ConnectButton, useCurrentAccount } from '@iota/dapp-kit';

function App() {
    const account = useCurrentAccount();
    return (
        <div>
            <ConnectButton />
            {account && <p>Connected: {account.address}</p>}
        </div>
    );
}
```

## 7. Migration from Firefly

Firefly was the wallet for legacy IOTA (Stardust/Chrysalis). It is **not compatible** with IOTA Rebased.

### Migration Path

Token migration from Stardust to Rebased was handled through a specific migration process:

1. **If you migrated during the official migration window:** Your IOTA tokens were automatically migrated to a Rebased address derived from your Stardust keys. Import your mnemonic into the new IOTA Wallet.

2. **If you have a Firefly mnemonic:** Import it into the new IOTA Wallet. If your tokens were part of the migration, they will appear at the derived Rebased address.

3. **If you missed the migration window:** Check [migration.md](migration.md) for the latest information on late migration options.

> **Firefly is deprecated** for IOTA Rebased. All users should transition to the new IOTA Wallet extension.

## 8. Hardware Wallets

### Ledger

Ledger support for IOTA Rebased is available:

- **Ledger Nano S Plus / Nano X / Stax** — supported via the IOTA app on Ledger
- Install the IOTA app from Ledger Live
- Connect Ledger to the IOTA Wallet browser extension

### Setup with Ledger

1. Install the **IOTA app** on your Ledger device via Ledger Live
2. Open the IOTA Wallet browser extension
3. Choose **Connect Hardware Wallet**
4. Follow the prompts to pair your Ledger
5. Your Ledger-derived address appears in the wallet

> **Tip:** Hardware wallets provide the highest security for large holdings. Your private keys never leave the device.

## 9. CLI Wallet

The IOTA CLI also functions as a wallet for developers:

```bash
# Generate a new address
iota client new-address --key-scheme ed25519

# List all addresses
iota keytool list

# Check active address
iota client active-address

# Switch active address
iota client switch --address 0x<ADDRESS>

# Check balance
iota client balance

# Send IOTA
iota client transfer-iota \
    --to 0x<RECIPIENT> \
    --iota-coin-object-id 0x<COIN_ID> \
    --amount 1000000000 \
    --gas-budget 10000000

# Request testnet tokens
iota client faucet
```

### Keystore Location

Keys are stored at `~/.iota/iota_config/iota.keystore` (encrypted).

## 10. Security Best Practices

### Mnemonic Security

- ✅ Write your mnemonic on paper (or metal backup)
- ✅ Store in a secure, fire-proof location
- ✅ Consider splitting across multiple locations
- ❌ Never store mnemonics in digital files, cloud storage, or screenshots
- ❌ Never share your mnemonic with anyone
- ❌ Never enter your mnemonic on any website (except the official wallet)

### Wallet Security

- ✅ Use a strong, unique password for the wallet
- ✅ Lock the wallet when not in use
- ✅ Use a hardware wallet (Ledger) for large holdings
- ✅ Verify transaction details before signing
- ✅ Only connect to trusted dApps
- ❌ Never approve transactions you don't understand
- ❌ Beware of phishing sites mimicking the IOTA Wallet

### Transaction Security

- ✅ Double-check recipient addresses before sending
- ✅ Start with a small test transaction for new addresses
- ✅ Review gas fees before confirming
- ❌ Never sign transactions from untrusted dApps
- ❌ Beware of airdrop scams requesting wallet connections

## References

- [IOTA Wallet Getting Started](https://docs.iota.org/users/iota-wallet/getting-started)
- [IOTA Wallet on Chrome Web Store](https://chromewebstore.google.com/detail/iota-wallet/iidjkmdceolghepehaaddojmnjnkkija)
- [CLI Client reference](https://docs.iota.org/developer/references/cli/client)
- [dApp Kit docs](https://docs.iota.org/developer/ts-sdk/dapp-kit/)
