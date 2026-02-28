# DeFi Patterns in Move on IOTA

## Table of Contents

- [Overview](#overview)
- [1. Creating a Custom Token (Coin)](#1-creating-a-custom-token-coin)
- [2. Coin Operations](#2-coin-operations)
- [3. Balance vs Coin](#3-balance-vs-coin)
- [4. NFTs on IOTA](#4-nfts-on-iota)
- [5. NFT Collections with Display](#5-nft-collections-with-display)
- [6. DEX / AMM Patterns](#6-dex--amm-patterns)
- [7. Flash Loans](#7-flash-loans)
- [8. Oracle Patterns](#8-oracle-patterns)
- [9. Sponsored Transactions](#9-sponsored-transactions)
- [References](#references)

## Overview

IOTA Rebased uses Move's type system to represent fungible tokens (`Coin<T>`), non-fungible tokens (objects with `key + store`), and DeFi primitives. The `iota::coin` module provides the standard for fungible tokens, while NFTs are simply unique objects.

## 1. Creating a Custom Token (Coin)

Every fungible token on IOTA uses the `Coin<T>` standard from `iota::coin`. You create one using a One-Time Witness (OTW):

```move
module examples::my_token {
    use iota::coin;
    use iota::url;

    /// OTW — must be module name in UPPERCASE, only `drop`.
    public struct MY_TOKEN has drop {}

    fun init(witness: MY_TOKEN, ctx: &mut TxContext) {
        let (treasury_cap, metadata) = coin::create_currency(
            witness,
            9,                              // decimals
            b"MYT",                         // symbol
            b"My Token",                    // name
            b"A custom token on IOTA",      // description
            option::some(url::new_unsafe_from_bytes(
                b"https://example.com/icon.png"
            )),                             // icon URL
            ctx,
        );

        // Freeze metadata — immutable, visible to all
        transfer::public_freeze_object(metadata);

        // Transfer TreasuryCap to deployer — controls minting
        transfer::public_transfer(treasury_cap, tx_context::sender(ctx));
    }
}
```

### Minting and Burning

```move
module examples::my_token {
    // ... (init above)

    /// Mint tokens — requires TreasuryCap.
    public fun mint(
        treasury: &mut TreasuryCap<MY_TOKEN>,
        amount: u64,
        recipient: address,
        ctx: &mut TxContext,
    ) {
        let coin = coin::mint(treasury, amount, ctx);
        transfer::public_transfer(coin, recipient);
    }

    /// Burn tokens — requires TreasuryCap.
    public fun burn(
        treasury: &mut TreasuryCap<MY_TOKEN>,
        coin: Coin<MY_TOKEN>,
    ) {
        coin::burn(treasury, coin);
    }
}
```

## 2. Coin Operations

The `iota::coin` module provides standard operations:

```move
use iota::coin::{Self, Coin};
use iota::iota::IOTA;

/// Split a coin — take `amount` out and return the new coin.
public fun take_payment(payment: &mut Coin<IOTA>, amount: u64, ctx: &mut TxContext): Coin<IOTA> {
    coin::split(payment, amount, ctx)
}

/// Merge two coins into one.
public fun combine(coin1: &mut Coin<IOTA>, coin2: Coin<IOTA>) {
    coin::join(coin1, coin2);
}

/// Check coin value.
public fun check_balance(coin: &Coin<IOTA>): u64 {
    coin::value(coin)
}

/// Create a zero-value coin.
public fun empty_coin(ctx: &mut TxContext): Coin<IOTA> {
    coin::zero(ctx)
}
```

**Best practice for accepting payments:**

```move
// ✅ Good — caller knows exactly what they pay
public fun buy_item(payment: Coin<IOTA>, ctx: &mut TxContext): Item {
    assert!(coin::value(&payment) == ITEM_PRICE, EIncorrectPayment);
    transfer::public_transfer(payment, @treasury);
    Item { id: object::new(ctx) }
}

// ❌ Bad — caller must trust the function to take the right amount
public fun buy_item(payment: &mut Coin<IOTA>, amount: u64, ctx: &mut TxContext): Item {
    let paid = coin::split(payment, amount, ctx);
    // ...
}
```

## 3. Balance vs Coin

`Coin<T>` is an object (has `key`). `Balance<T>` is a raw value (has `store` only) — lighter weight for internal accounting.

```move
use iota::balance::{Self, Balance};
use iota::coin::{Self, Coin};

public struct Vault has key {
    id: UID,
    funds: Balance<IOTA>,    // Not a separate object — stored inline
}

public fun deposit(vault: &mut Vault, payment: Coin<IOTA>) {
    let bal = coin::into_balance(payment);  // Coin → Balance
    balance::join(&mut vault.funds, bal);
}

public fun withdraw(vault: &mut Vault, amount: u64, ctx: &mut TxContext): Coin<IOTA> {
    let bal = balance::split(&mut vault.funds, amount);
    coin::from_balance(bal, ctx)            // Balance → Coin
}

public fun total(vault: &Vault): u64 {
    balance::value(&vault.funds)
}
```

**When to use which:**
- `Coin<T>` — when transferring to users, accepting payments, public-facing
- `Balance<T>` — for internal pool balances, vaults, protocol reserves

## 4. NFTs on IOTA

NFTs are simply unique objects. There's no special "NFT standard" — any struct with `key` (and optionally `store`) is an NFT:

```move
module examples::basic_nft {
    use std::string::String;

    public struct ArtNFT has key, store {
        id: UID,
        name: String,
        description: String,
        image_url: String,
        creator: address,
    }

    public fun mint(
        name: String,
        description: String,
        image_url: String,
        ctx: &mut TxContext,
    ): ArtNFT {
        ArtNFT {
            id: object::new(ctx),
            name,
            description,
            image_url,
            creator: tx_context::sender(ctx),
        }
    }

    /// Burn (destroy) the NFT.
    public fun burn(nft: ArtNFT) {
        let ArtNFT { id, name: _, description: _, image_url: _, creator: _ } = nft;
        object::delete(id);
    }
}
```

## 5. NFT Collections with Display

The `Display` standard tells wallets and explorers how to render your objects:

```move
module examples::collectible {
    use std::string::String;
    use iota::package;
    use iota::display;

    public struct COLLECTIBLE has drop {}

    public struct Card has key, store {
        id: UID,
        name: String,
        image_url: String,
        power: u64,
    }

    fun init(otw: COLLECTIBLE, ctx: &mut TxContext) {
        let publisher = package::claim(otw, ctx);

        let mut disp = display::new_with_fields<Card>(
            &publisher,
            vector[
                b"name".to_string(),
                b"image_url".to_string(),
                b"description".to_string(),
            ],
            vector[
                b"{name}".to_string(),
                b"{image_url}".to_string(),
                b"Card with power {power}".to_string(),
            ],
            ctx,
        );
        display::update_version(&mut disp);

        transfer::public_transfer(publisher, tx_context::sender(ctx));
        transfer::public_transfer(disp, tx_context::sender(ctx));
    }

    public fun mint(name: String, image_url: String, power: u64, ctx: &mut TxContext): Card {
        Card { id: object::new(ctx), name, image_url, power }
    }
}
```

## 6. DEX / AMM Patterns

A simplified constant-product AMM on IOTA:

```move
module examples::simple_amm {
    use iota::balance::{Self, Balance};
    use iota::coin::{Self, Coin};

    const EInsufficientLiquidity: u64 = 0;
    const EZeroAmount: u64 = 1;

    /// LP token for this pool.
    public struct LP<phantom A, phantom B> has drop {}

    /// The AMM pool — shared object.
    public struct Pool<phantom A, phantom B> has key {
        id: UID,
        reserve_a: Balance<A>,
        reserve_b: Balance<B>,
        lp_supply: u64,
    }

    /// Create a new pool with initial liquidity.
    public fun create_pool<A, B>(
        coin_a: Coin<A>,
        coin_b: Coin<B>,
        ctx: &mut TxContext,
    ) {
        let bal_a = coin::into_balance(coin_a);
        let bal_b = coin::into_balance(coin_b);
        let lp_supply = balance::value(&bal_a); // simplified

        let pool = Pool<A, B> {
            id: object::new(ctx),
            reserve_a: bal_a,
            reserve_b: bal_b,
            lp_supply,
        };
        transfer::share_object(pool);
    }

    /// Swap A for B using constant product formula.
    public fun swap_a_for_b<A, B>(
        pool: &mut Pool<A, B>,
        coin_in: Coin<A>,
        ctx: &mut TxContext,
    ): Coin<B> {
        let amount_in = coin::value(&coin_in);
        assert!(amount_in > 0, EZeroAmount);

        let reserve_a = balance::value(&pool.reserve_a);
        let reserve_b = balance::value(&pool.reserve_b);

        // Constant product: (reserve_a + amount_in) * (reserve_b - amount_out) = reserve_a * reserve_b
        // With 0.3% fee:
        let amount_in_with_fee = amount_in * 997;
        let numerator = amount_in_with_fee * reserve_b;
        let denominator = (reserve_a * 1000) + amount_in_with_fee;
        let amount_out = numerator / denominator;

        assert!(amount_out > 0 && amount_out < reserve_b, EInsufficientLiquidity);

        balance::join(&mut pool.reserve_a, coin::into_balance(coin_in));
        let out_balance = balance::split(&mut pool.reserve_b, amount_out);
        coin::from_balance(out_balance, ctx)
    }
}
```

> **Note:** This is a simplified example. Production AMMs need LP token minting/burning, slippage protection, fee collection, and proper math (sqrt for initial LP).

## 7. Flash Loans

Flash loans use the [Hot Potato pattern](move-advanced.md#5-hot-potato-pattern) to enforce repayment within the same transaction:

```move
module examples::flash_lender {
    use iota::balance::{Self, Balance};
    use iota::coin::{Self, Coin};

    const EInsufficientFunds: u64 = 0;
    const ERepaymentTooLow: u64 = 1;

    public struct FlashLender<phantom T> has key {
        id: UID,
        funds: Balance<T>,
        fee_bps: u64,       // Fee in basis points (100 = 1%)
    }

    /// Hot potato — must be consumed by `repay` in the same tx.
    public struct FlashReceipt<phantom T> {
        lender_id: ID,
        amount: u64,
        fee: u64,
    }

    /// Borrow funds — returns coins + a receipt (hot potato).
    public fun borrow<T>(
        lender: &mut FlashLender<T>,
        amount: u64,
        ctx: &mut TxContext,
    ): (Coin<T>, FlashReceipt<T>) {
        assert!(balance::value(&lender.funds) >= amount, EInsufficientFunds);

        let fee = (amount * lender.fee_bps) / 10000;
        let loan = balance::split(&mut lender.funds, amount);

        let receipt = FlashReceipt<T> {
            lender_id: object::id(lender),
            amount,
            fee,
        };

        (coin::from_balance(loan, ctx), receipt)
    }

    /// Repay the loan — consumes the receipt (hot potato resolved).
    public fun repay<T>(
        lender: &mut FlashLender<T>,
        receipt: FlashReceipt<T>,
        repayment: Coin<T>,
    ) {
        let FlashReceipt { lender_id, amount, fee } = receipt;
        assert!(object::id(lender) == lender_id, 0);
        assert!(coin::value(&repayment) >= amount + fee, ERepaymentTooLow);

        balance::join(&mut lender.funds, coin::into_balance(repayment));
    }
}
```

## 8. Oracle Patterns

Oracles on IOTA typically use shared objects updated by authorized feeds:

```move
module examples::price_oracle {
    const ENotAuthorized: u64 = 0;
    const EStalePrice: u64 = 1;

    public struct OracleAdminCap has key, store { id: UID }

    public struct PriceFeed has key {
        id: UID,
        price: u64,           // Price in smallest unit
        decimals: u8,
        last_updated: u64,    // Epoch timestamp
    }

    fun init(ctx: &mut TxContext) {
        transfer::transfer(
            OracleAdminCap { id: object::new(ctx) },
            tx_context::sender(ctx),
        );

        transfer::share_object(PriceFeed {
            id: object::new(ctx),
            price: 0,
            decimals: 8,
            last_updated: 0,
        });
    }

    /// Update price — requires admin capability.
    public fun update_price(
        _admin: &OracleAdminCap,
        feed: &mut PriceFeed,
        new_price: u64,
        ctx: &TxContext,
    ) {
        feed.price = new_price;
        feed.last_updated = tx_context::epoch(ctx);
    }

    /// Read price — consumers check staleness themselves.
    public fun get_price(feed: &PriceFeed): (u64, u8, u64) {
        (feed.price, feed.decimals, feed.last_updated)
    }
}
```

## 9. Sponsored Transactions

IOTA Rebased supports **sponsored transactions** where an app pays gas fees on behalf of users. This is handled at the transaction level (not in Move code) using the SDK:

```typescript
// TypeScript SDK example
import { IotaClient } from '@iota/iota-sdk/client';
import { Transaction } from '@iota/iota-sdk/transactions';

const tx = new Transaction();
tx.setSender(userAddress);
tx.setGasOwner(sponsorAddress);   // Sponsor pays gas
tx.setGasBudget(10_000_000);

// User signs the tx, then sponsor signs separately
const userSig = await userKeypair.signTransaction(txBytes);
const sponsorSig = await sponsorKeypair.signTransaction(txBytes);

// Execute with both signatures
await client.executeTransactionBlock({
    transactionBlock: txBytes,
    signature: [userSig.signature, sponsorSig.signature],
});
```

This enables gasless UX for end users — critical for mainstream adoption.

## 10. Closed-Loop Tokens (`iota::token`)

Unlike `Coin<T>` which is freely transferable, `Token<T>` is a closed-loop token with spend policies enforced by the creator. Useful for loyalty points, in-game currency, vouchers.

```move
module examples::loyalty {
    use iota::token::{Self, Token, ActionRequest};
    use iota::coin::{Self, TreasuryCap};

    public struct LOYALTY has drop {}

    fun init(witness: LOYALTY, ctx: &mut TxContext) {
        let (treasury_cap, metadata) = coin::create_currency(
            witness, 0, b"LOYALTY", b"Loyalty Points",
            b"Non-transferable loyalty points", option::none(), ctx,
        );
        transfer::public_freeze_object(metadata);

        // Create token policy — controls what actions are allowed
        let (mut policy, policy_cap) = token::new_policy(&treasury_cap, ctx);

        // Allow spending but not peer-to-peer transfer
        token::allow(&mut policy, &policy_cap, token::spend_action(), ctx);

        token::share_policy(policy);
        transfer::public_transfer(policy_cap, ctx.sender());
        transfer::public_transfer(treasury_cap, ctx.sender());
    }
}
```

## 11. Kiosk (Commerce Standard)

The `iota::kiosk` module provides marketplace primitives — list, purchase, and enforce royalties:

```move
use iota::kiosk::{Self, Kiosk, KioskOwnerCap};
use iota::transfer_policy::{Self, TransferPolicy};

// Create a kiosk (marketplace storefront)
public fun create_kiosk(ctx: &mut TxContext): (Kiosk, KioskOwnerCap) {
    kiosk::new(ctx)
}

// List an item for sale
public fun list_item<T: key + store>(
    kiosk: &mut Kiosk,
    cap: &KioskOwnerCap,
    item_id: ID,
    price: u64,
) {
    kiosk::list<T>(kiosk, cap, item_id, price);
}
```

## References

- [IOTA Coin module](https://docs.iota.org/developer/references/framework/iota/coin)
- [Balance module](https://docs.iota.org/developer/references/framework/iota/balance)
- [Display standard](https://docs.iota.org/developer/standards/display)
- [Token transfer tutorial](https://docs.iota.org/developer/tutorials/simple-token-transfer)
- [Flash Loan example](https://github.com/iotaledger/iota/blob/develop/examples/move/flash_lender/sources/example.move)
- [← Prev: Objects & Fields](move-objects-fields.md) | [→ Next: Testing](move-testing.md)
