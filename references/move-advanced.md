# Advanced Move Patterns on IOTA

## Table of Contents

- [Overview](#overview)
- [1. Capability Pattern](#1-capability-pattern)
- [2. Witness Pattern](#2-witness-pattern)
- [3. One-Time Witness (OTW)](#3-one-time-witness-otw)
- [4. Transferable Witness](#4-transferable-witness)
- [5. Hot Potato Pattern](#5-hot-potato-pattern)
- [6. ID Pointer Pattern](#6-id-pointer-pattern)
- [7. Publisher Pattern](#7-publisher-pattern)
- [8. Generics and Type Constraints](#8-generics-and-type-constraints)
- [9. Move 2024 Edition Features](#9-move-2024-edition-features)
- [References](#references)

## Overview

This document covers advanced design patterns used in Move smart contracts on IOTA Rebased. These patterns leverage Move's type system and abilities (`key`, `store`, `copy`, `drop`) to enforce access control, ensure single-use semantics, and build composable protocols.

> **IOTA-specific**: All examples use `iota::` framework modules. IOTA Rebased is based on Sui's protocol with IOTA-specific improvements, so most Move patterns from Sui apply directly but with `iota::` namespaces.

## 1. Capability Pattern

A **capability** is an object that grants its owner authorization to perform specific actions. The holder of the capability can perform the privileged operation; without it, they cannot.

**Common examples in IOTA:**
- `TreasuryCap` — authority to mint coins
- `UpgradeCap` — authority to upgrade packages
- `AdminCap` — custom admin access

```move
module examples::item {
    use std::string::String;

    /// Capability object — only the holder can create items.
    public struct AdminCap has key { id: UID }

    /// An NFT-like item.
    public struct Item has key, store { id: UID, name: String }

    /// On publish, send AdminCap to the deployer.
    fun init(ctx: &mut TxContext) {
        transfer::transfer(AdminCap {
            id: object::new(ctx)
        }, tx_context::sender(ctx))
    }

    /// Requires AdminCap reference to authorize creation.
    public fun create_item(_: &AdminCap, name: String, ctx: &mut TxContext): Item {
        Item { id: object::new(ctx), name }
    }
}
```

**Key points:**
- The capability is an owned object (`has key`) — only its owner can pass it.
- Using `&AdminCap` (reference) lets the caller keep the capability after the call.
- You can make capabilities transferable by adding `store` ability.
- You can create multi-tier access (e.g., `AdminCap`, `ModeratorCap`) for role-based control.

### Revoking Capabilities

To make capabilities revocable, store them in a shared object or use a registry:

```move
module examples::revocable_admin {
    public struct AdminCap has key, store { id: UID }

    public struct AdminRegistry has key {
        id: UID,
        active_admins: vector<address>,
    }

    public fun revoke_admin(registry: &mut AdminRegistry, cap: AdminCap) {
        let AdminCap { id } = cap;
        object::delete(id);
        // Remove from active_admins vector...
    }
}
```

## 2. Witness Pattern

A **witness** is a type with `drop` ability used to prove the caller is authorized. It works because only code inside the defining module can instantiate it.

```move
/// Generic Guardian that can only be created with a witness.
module examples::guardian {
    public struct Guardian<phantom T: drop> has key, store {
        id: UID
    }

    /// The witness T is consumed (dropped) — proving the caller's module created it.
    public fun create_guardian<T: drop>(
        _witness: T, ctx: &mut TxContext
    ): Guardian<T> {
        Guardian { id: object::new(ctx) }
    }
}

/// A module that uses the Guardian with its own witness.
module examples::peace_guardian {
    use examples::guardian;

    /// Only this module can create PEACE instances.
    public struct PEACE has drop {}

    fun init(ctx: &mut TxContext) {
        transfer::public_transfer(
            guardian::create_guardian(PEACE {}, ctx),
            tx_context::sender(ctx)
        )
    }
}
```

**Key points:**
- The `phantom` keyword means the type parameter `T` doesn't exist at runtime — it's only used for type checking.
- The witness proves origin: only the defining module can instantiate `PEACE`.
- Combined with `init`, this guarantees one-time-only execution.

## 3. One-Time Witness (OTW)

A **One-Time Witness** is a special type that is:
- Named after the module in UPPERCASE
- Has only the `drop` ability
- Automatically passed to `init()` on publish — **only one instance ever exists**

The IOTA runtime enforces these properties via `iota::types::is_one_time_witness`.

```move
module examples::my_otw {
    use std::string;
    use iota::types;

    const ENotOneTimeWitness: u64 = 0;

    /// OTW: uppercase module name, only `drop`.
    public struct MY_OTW has drop {}

    /// Automatically receives the OTW instance.
    fun init(witness: MY_OTW, ctx: &mut TxContext) {
        // Verify it's a genuine OTW
        assert!(types::is_one_time_witness(&witness), ENotOneTimeWitness);

        // Use it for one-time operations like creating a coin...
    }
}
```

**OTW rules (enforced by IOTA runtime):**
1. Type name = module name in UPPERCASE
2. Has only `drop` ability (no `store`, no `copy`, no `key`)
3. Has no fields, OR only `bool` field with value `false`
4. Created only once — in the `init` function parameter

**Common use case**: Creating a coin type (the `Coin` module requires an OTW):

```move
module examples::my_coin {
    use iota::coin;

    public struct MY_COIN has drop {}

    fun init(witness: MY_COIN, ctx: &mut TxContext) {
        let (treasury_cap, metadata) = coin::create_currency(
            witness,
            9,                          // decimals
            b"MYC",                     // symbol
            b"My Coin",                 // name
            b"A custom IOTA token",     // description
            option::none(),             // icon URL
            ctx,
        );
        transfer::public_freeze_object(metadata);
        transfer::public_transfer(treasury_cap, tx_context::sender(ctx));
    }
}
```

## 4. Transferable Witness

Combines **Capability** + **Witness** patterns. Useful when authorization needs to be delegated across modules or deferred over time.

```move
module examples::transferable_witness {

    /// A storable, droppable witness.
    public struct WITNESS has store, drop {}

    /// A carrier that can be transferred to other addresses.
    public struct WitnessCarrier has key {
        id: UID,
        witness: WITNESS
    }

    /// On publish, send the carrier to the deployer.
    fun init(ctx: &mut TxContext) {
        transfer::transfer(
            WitnessCarrier {
                id: object::new(ctx),
                witness: WITNESS {}
            },
            tx_context::sender(ctx)
        )
    }

    /// Extract and consume the witness — single-use.
    public fun get_witness(carrier: WitnessCarrier): WITNESS {
        let WitnessCarrier { id, witness } = carrier;
        object::delete(id);
        witness
    }
}
```

**Use cases:**
- Cross-module authorization: Module A creates the witness, Module B consumes it
- Deferred authorization: Create now, use later when conditions are met
- Single-use access tokens: The carrier is destroyed on extraction

## 5. Hot Potato Pattern

A **hot potato** is a struct with **no abilities** — it cannot be stored, copied, dropped, or transferred. It **must** be consumed (unpacked) in the same transaction it was created.

This forces the caller to complete a specific workflow within a single transaction.

```move
module examples::trade_in {
    use iota::iota::IOTA;
    use iota::coin::{Self, Coin};

    const MODEL_ONE_PRICE: u64 = 10000;
    const MODEL_TWO_PRICE: u64 = 20000;
    const EWrongModel: u64 = 1;
    const EIncorrectAmount: u64 = 2;

    public struct Phone has key, store { id: UID, model: u8 }

    /// HOT POTATO: no abilities — must be consumed in the same tx.
    public struct Receipt { price: u64 }

    /// Step 1: Get a phone + receipt (hot potato).
    public fun buy_phone(model: u8, ctx: &mut TxContext): (Phone, Receipt) {
        assert!(model == 1 || model == 2, EWrongModel);
        let price = if (model == 1) MODEL_ONE_PRICE else MODEL_TWO_PRICE;
        (
            Phone { id: object::new(ctx), model },
            Receipt { price }
        )
    }

    /// Step 2a: Pay full price — consumes the Receipt.
    public fun pay_full(receipt: Receipt, payment: Coin<IOTA>) {
        let Receipt { price } = receipt;
        assert!(coin::value(&payment) == price, EIncorrectAmount);
        transfer::public_transfer(payment, @examples);
    }

    /// Step 2b: Trade in old phone for discount — consumes the Receipt.
    public fun trade_in(receipt: Receipt, old_phone: Phone, payment: Coin<IOTA>) {
        let Receipt { price } = receipt;
        let tradein_price = if (old_phone.model == 1) MODEL_ONE_PRICE else MODEL_TWO_PRICE;
        let to_pay = price - (tradein_price / 2);
        assert!(coin::value(&payment) == to_pay, EIncorrectAmount);
        transfer::public_transfer(old_phone, @examples);
        transfer::public_transfer(payment, @examples);
    }
}
```

**Classic use case — Flash Loans:**
1. `borrow()` returns funds + a `Receipt` (hot potato)
2. Caller uses the funds however they want
3. `repay()` requires the `Receipt` + repayment — enforces repayment in same tx
4. If `repay()` isn't called, the tx aborts (Receipt can't be dropped)

See the [Flash Loan example](https://github.com/iotaledger/iota/blob/develop/examples/move/flash_lender/sources/example.move) in the IOTA repo.

## 6. ID Pointer Pattern

Objects can reference each other using their on-chain IDs. This decouples objects while maintaining relationships.

```move
module examples::id_pointer {
    use iota::object::ID;

    public struct Vault has key {
        id: UID,
        balance: u64,
    }

    /// Points to a Vault by its ID — doesn't own or wrap it.
    public struct VaultAccess has key, store {
        id: UID,
        vault_id: ID,
    }

    public fun create_vault(ctx: &mut TxContext): (Vault, VaultAccess) {
        let vault = Vault { id: object::new(ctx), balance: 0 };
        let vault_id = object::id(&vault);
        let access = VaultAccess { id: object::new(ctx), vault_id };
        (vault, access)
    }

    /// Authorization check via ID matching.
    public fun deposit(vault: &mut Vault, access: &VaultAccess, amount: u64) {
        assert!(object::id(vault) == access.vault_id, 0);
        vault.balance = vault.balance + amount;
    }
}
```

**Advantages over wrapping:**
- Referenced object stays independently accessible (not wrapped/hidden)
- Multiple pointers can reference the same object
- Useful for NFT collections, access control, linked data

## 7. Publisher Pattern

The `Publisher` object is created using `iota::package::claim` with an OTW. It proves you published a specific package.

```move
module examples::my_nft {
    use iota::package;
    use iota::display;
    use std::string::String;

    public struct MY_NFT has drop {}

    public struct NFT has key, store {
        id: UID,
        name: String,
        image_url: String,
    }

    fun init(otw: MY_NFT, ctx: &mut TxContext) {
        let publisher = package::claim(otw, ctx);

        // Use Publisher to create a Display template
        let mut disp = display::new_with_fields<NFT>(
            &publisher,
            vector[
                b"name".to_string(),
                b"image_url".to_string(),
            ],
            vector[
                b"{name}".to_string(),
                b"{image_url}".to_string(),
            ],
            ctx,
        );
        display::update_version(&mut disp);

        transfer::public_transfer(publisher, tx_context::sender(ctx));
        transfer::public_transfer(disp, tx_context::sender(ctx));
    }
}
```

**Publisher enables:**
- Creating `Display` objects (customize how your types appear in wallets/explorers)
- Proving package authorship on-chain
- Setting up transfer policies for regulated assets

## 8. Generics and Type Constraints

Move generics with ability constraints are fundamental to all patterns above.

### Ability Quick Reference

| Ability | Meaning | Example |
|---------|---------|---------|
| `key` | Can be stored as an object on-chain (requires `id: UID` first field) | Objects, NFTs |
| `store` | Can be stored inside other objects | Transferable assets |
| `copy` | Can be duplicated | Integers, booleans |
| `drop` | Can be silently discarded | Witnesses, receipts |

### Phantom Types

Use `phantom` for type parameters not used in the struct's fields:

```move
/// T is phantom — used only for type-level distinction, not stored.
public struct Coin<phantom T> has key, store {
    id: UID,
    balance: Balance<T>,  // Balance uses T, but Coin's T is phantom
}
```

### Constraint Examples

```move
/// Accept any type that can be stored.
public fun wrap<T: store>(item: T, ctx: &mut TxContext): Wrapper<T> {
    Wrapper { id: object::new(ctx), item }
}

/// Accept only witnesses (types with drop).
public fun verified_action<T: drop>(_witness: T) {
    // Only code that can create T can call this
}
```

## 9. Move 2024 Edition Features

IOTA launches with Move 2024, which adds significant syntax improvements:

### Method Syntax
```move
// Old: vector::push_back(&mut v, coin::value(&c));
// New:
v.push_back(c.value());
```

### Enums and Pattern Matching
```move
public enum Option<T> {
    None,
    Some(T),
}

public fun unwrap<T>(opt: Option<T>): T {
    match (opt) {
        Option::Some(x) => x,
        Option::None => abort 0,
    }
}
```

### Macro Functions
```move
let doubled = v.map!(|x| x * 2);
v.for_each!(|item| process(item));
```

### Index Syntax
```move
// v[i] desugars to *vector::borrow(&v, i)
let val = v[i];
*&mut v[i] = new_val;
```

### Other Improvements
- `public(package)` replaces `public(friend)`
- `mut` required for mutable variables: `let mut x = 0;`
- Positional struct fields: `public struct Pair(u64, u64) has copy, drop, store;`
- Loop labels: `'outer: while (cond) { ... break 'outer; }`
- `break` with value: `let x = loop { if (cond) break val; };`
- Type inference holes: `borrow_mut<_, Coin<IOTA>>(&mut id, owner)`
- Nested `use`: `use iota::{balance, coin::{Self, Coin}};`

### Auto-imported in Every Module (Move 2024)
```move
use std::vector;
use std::option::{Self, Option};
use iota::object::{Self, ID, UID};
use iota::transfer;
use iota::tx_context::{Self, TxContext};
```

## References

- [IOTA Patterns docs](https://docs.iota.org/developer/iota-101/move-overview/patterns/)
- [One-Time Witness](https://docs.iota.org/developer/iota-101/move-overview/one-time-witness)
- [Move 2024 edition](https://docs.iota.org/developer/advanced/introducing-move-2024)
- [Flash Loan example](https://github.com/iotaledger/iota/blob/develop/examples/move/flash_lender/sources/example.move)
- [Hero example](https://github.com/iotaledger/iota/blob/develop/examples/move/hero/sources/example.move)
- [→ Next: Objects & Dynamic Fields](move-objects-fields.md)
