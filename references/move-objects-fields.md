# Objects and Dynamic Fields on IOTA

## Table of Contents

- [Overview](#overview)
- [1. The Object Model](#1-the-object-model)
- [2. Object Ownership Types](#2-object-ownership-types)
- [3. Creating and Deleting Objects](#3-creating-and-deleting-objects)
- [4. Wrapping Objects](#4-wrapping-objects)
- [5. Dynamic Fields](#5-dynamic-fields)
- [6. Dynamic Object Fields](#6-dynamic-object-fields)
- [7. Tables and Bags](#7-tables-and-bags)
- [8. Transfer and Freeze](#8-transfer-and-freeze)
- [9. Best Practices](#9-best-practices)
- [References](#references)

## Overview

In IOTA Rebased, **everything is an object**. Unlike account-based blockchains (Ethereum) or UTXO models (Bitcoin, old IOTA Stardust), IOTA Rebased uses an **object-centric model** where each asset has a globally unique ID and defined ownership.

## 1. The Object Model

An object in Move is any struct with the `key` ability and `id: UID` as its first field:

```move
public struct MyObject has key, store {
    id: UID,          // Required first field — globally unique ID
    value: u64,
    name: String,
}
```

**Object properties:**
- Every object has a unique 32-byte address (its `id`)
- Objects are the unit of storage — they live on-chain independently
- Objects track their own version (incremented on every mutation)
- Maximum object size: **250 KB**

## 2. Object Ownership Types

| Ownership | Description | Tx Ordering | Use Case |
|-----------|-------------|-------------|----------|
| **Owned** | Belongs to a specific address | Parallel (no consensus needed) | Personal assets, capabilities |
| **Shared** | Accessible by anyone | Sequential (requires consensus) | AMM pools, shared registries |
| **Immutable** | Frozen forever, read-only | Parallel | Package metadata, configs |
| **Wrapped** | Stored inside another object | Via parent object | Composition, bundling |

### Owned Objects

```move
/// Transfer to a specific address — only that address can use it.
public fun create_and_send(recipient: address, ctx: &mut TxContext) {
    let obj = MyObject {
        id: object::new(ctx),
        value: 100,
        name: b"hello".to_string(),
    };
    transfer::transfer(obj, recipient);
}
```

**Owned object transactions execute in parallel** — this is IOTA's key scalability feature. If two transactions touch different owned objects, they don't need consensus ordering.

### Shared Objects

```move
/// Share an object — anyone can read/write it (with consensus ordering).
public fun create_shared_pool(ctx: &mut TxContext) {
    let pool = LiquidityPool {
        id: object::new(ctx),
        balance_a: balance::zero(),
        balance_b: balance::zero(),
    };
    transfer::share_object(pool);
}
```

> **Performance note:** Shared objects require consensus ordering, which is slower than owned objects. Use shared objects only when multiple users must write to the same object.

### Immutable Objects

```move
/// Freeze an object forever — no one can modify it.
public fun freeze_config(config: GameConfig) {
    transfer::freeze_object(config);
}

// Or with public_ prefix for objects with `store`:
transfer::public_freeze_object(metadata);
```

## 3. Creating and Deleting Objects

### Creating

```move
public fun new_item(ctx: &mut TxContext): Item {
    Item {
        id: object::new(ctx),    // Generates unique ID
        data: 42,
    }
}
```

### Deleting

To delete an object, destructure it and call `object::delete` on the UID:

```move
public fun destroy_item(item: Item) {
    let Item { id, data: _ } = item;
    object::delete(id);
}
```

> **Important:** You can only delete an object inside the module that defines it (because only that module can destructure it).

### Getting Object Info

```move
// Get the object's ID
let id: ID = object::id(&my_obj);

// Get the object's address
let addr: address = object::id_address(&my_obj);

// Get the UID reference
let uid: &UID = object::uid(&my_obj);
```

## 4. Wrapping Objects

An object can be stored inside another object (wrapping). The wrapped object becomes **inaccessible by ID** — it can only be reached through its parent.

```move
public struct Wrapper has key {
    id: UID,
    inner: InnerObject,    // InnerObject is wrapped — hidden from explorers
}

public struct InnerObject has key, store {
    id: UID,
    value: u64,
}

public fun wrap(inner: InnerObject, ctx: &mut TxContext): Wrapper {
    Wrapper { id: object::new(ctx), inner }
}

public fun unwrap(wrapper: Wrapper): InnerObject {
    let Wrapper { id, inner } = wrapper;
    object::delete(id);
    inner
}
```

**When to wrap vs. use dynamic fields:**
- **Wrap** when the relationship is permanent and the inner object is always needed
- **Dynamic fields** when you want flexible, on-demand attachment

## 5. Dynamic Fields

Dynamic fields let you add/remove fields from an object at runtime. Unlike struct fields (fixed at publish time), dynamic fields are flexible and only cost gas when accessed.

### API

```move
module iota::dynamic_field {
    /// Add a field. Aborts if a field with this name already exists.
    public fun add<Name: copy + drop + store, Value: store>(
        object: &mut UID, name: Name, value: Value,
    );

    /// Borrow a field by reference.
    public fun borrow<Name: copy + drop + store, Value: store>(
        object: &UID, name: Name,
    ): &Value;

    /// Borrow a field mutably.
    public fun borrow_mut<Name: copy + drop + store, Value: store>(
        object: &mut UID, name: Name,
    ): &mut Value;

    /// Remove a field and return its value.
    public fun remove<Name: copy + drop + store, Value: store>(
        object: &mut UID, name: Name,
    ): Value;

    /// Check if a field exists.
    public fun exists_<Name: copy + drop + store>(
        object: &UID, name: Name,
    ): bool;
}
```

### Example

```move
module examples::dynamic_example {
    use iota::dynamic_field as df;

    public struct Registry has key {
        id: UID,
    }

    /// Add a key-value pair to the registry.
    public fun set(registry: &mut Registry, key: String, value: u64) {
        if (df::exists_(&registry.id, key)) {
            *df::borrow_mut(&mut registry.id, key) = value;
        } else {
            df::add(&mut registry.id, key, value);
        };
    }

    /// Read a value.
    public fun get(registry: &Registry, key: String): u64 {
        *df::borrow(&registry.id, key)
    }

    /// Remove a value.
    public fun remove(registry: &mut Registry, key: String): u64 {
        df::remove(&mut registry.id, key)
    }
}
```

**Dynamic field names** can be any type with `copy + drop + store` — not just strings. You can use integers, addresses, or custom structs:

```move
// Using an integer key
df::add(&mut obj.id, 42u64, some_value);

// Using a custom struct key
public struct SlotKey has copy, drop, store { slot: u8 }
df::add(&mut obj.id, SlotKey { slot: 1 }, item);
```

## 6. Dynamic Object Fields

Dynamic **object** fields are similar but require the value to be an object (`key + store`). The key difference: **the stored object remains accessible by its ID** through explorers and wallets.

```move
module examples::parent_child {
    use iota::dynamic_object_field as ofield;

    public struct Parent has key {
        id: UID,
    }

    public struct Child has key, store {
        id: UID,
        count: u64,
    }

    /// Add Child as a dynamic object field of Parent.
    public fun add_child(parent: &mut Parent, child: Child) {
        ofield::add(&mut parent.id, b"child", child);
    }

    /// Mutate child through parent.
    public fun increment_child(parent: &mut Parent) {
        let child: &mut Child = ofield::borrow_mut(&mut parent.id, b"child");
        child.count = child.count + 1;
    }

    /// Remove child from parent.
    public fun detach_child(parent: &mut Parent): Child {
        ofield::remove(&mut parent.id, b"child")
    }
}
```

### Dynamic Fields vs. Dynamic Object Fields

| Feature | `dynamic_field` | `dynamic_object_field` |
|---------|----------------|----------------------|
| Value requirement | `store` | `key + store` |
| Value accessible by ID? | No (wrapped) | Yes (visible in explorers) |
| Use when | Storing primitives, non-object data | Storing objects you want externally visible |

## 7. Tables and Bags

IOTA provides collection types built on dynamic fields with safety features (e.g., preventing accidental deletion when non-empty):

### Table (homogeneous)

```move
use iota::table::{Self, Table};

public struct GameState has key {
    id: UID,
    scores: Table<address, u64>,   // All keys same type, all values same type
}

public fun init_game(ctx: &mut TxContext) {
    let state = GameState {
        id: object::new(ctx),
        scores: table::new(ctx),
    };
    transfer::share_object(state);
}

public fun set_score(state: &mut GameState, player: address, score: u64) {
    if (table::contains(&state.scores, player)) {
        *table::borrow_mut(&mut state.scores, player) = score;
    } else {
        table::add(&mut state.scores, player, score);
    };
}

public fun get_score(state: &GameState, player: address): u64 {
    *table::borrow(&state.scores, player)
}
```

### Bag (heterogeneous)

```move
use iota::bag::{Self, Bag};

public struct Inventory has key {
    id: UID,
    items: Bag,    // Can store different types under different keys
}

public fun add_sword(inv: &mut Inventory, sword: Sword) {
    bag::add(&mut inv.items, b"sword", sword);
}

public fun add_gold(inv: &mut Inventory, amount: u64) {
    bag::add(&mut inv.items, b"gold", amount);
}
```

### Collection Types Summary

| Type | Keys | Values | Object values visible by ID? |
|------|------|--------|------------------------------|
| `Table<K, V>` | Homogeneous | Homogeneous | No |
| `ObjectTable<K, V>` | Homogeneous | Homogeneous objects | Yes |
| `Bag` | Heterogeneous | Heterogeneous | No |
| `ObjectBag` | Heterogeneous | Heterogeneous objects | Yes |
| `LinkedTable<K, V>` | Homogeneous | Homogeneous + iteration | No |
| `VecMap<K, V>` | Vector-backed | Vector-backed | N/A (in-object) |

> **Rule of thumb:** Use vector-backed collections (`vector`, `VecSet`, `VecMap`) for ≤ 1000 items with known max size. Use dynamic field-backed collections (`Table`, `Bag`) for unbounded or third-party-contributed collections.

## 8. Transfer and Freeze

### Transfer Functions

```move
// For types WITHOUT `store` — can only be called in the defining module:
transfer::transfer(obj, recipient);
transfer::share_object(obj);
transfer::freeze_object(obj);

// For types WITH `store` — can be called from any module:
transfer::public_transfer(obj, recipient);
transfer::public_share_object(obj);
transfer::public_freeze_object(obj);
```

### Custom Transfer Policies

By omitting `store` from your type, you restrict transfers to your module only — enabling custom transfer logic:

```move
public struct SoulboundNFT has key {
    id: UID,
    owner: address,
    // No `store` — cannot be transferred by anyone outside this module
}

/// Only allow transfer if a condition is met.
public fun conditional_transfer(nft: SoulboundNFT, new_owner: address) {
    // Custom logic here...
    transfer::transfer(nft, new_owner);
}
```

### Composability Tip

**Return objects from functions instead of self-transferring:**

```move
// ❌ Bad — breaks composability
public fun mint(ctx: &mut TxContext) {
    let nft = NFT { id: object::new(ctx), data: 0 };
    transfer::transfer(nft, tx_context::sender(ctx));
}

// ✅ Good — caller or PTB can decide what to do with it
public fun mint(ctx: &mut TxContext): NFT {
    NFT { id: object::new(ctx), data: 0 }
}
```

## 9. Best Practices

1. **Prefer owned objects** for user assets — they enable parallel execution
2. **Use shared objects sparingly** — they bottleneck throughput
3. **Max object size is 250 KB** — use dynamic fields for growing collections
4. **Don't use vector for unbounded collections** — use Table/Bag instead
5. **Return objects** from functions for composability with PTBs
6. **Use `public_transfer`/`public_freeze_object`** for types with `store`; `transfer`/`freeze_object` for module-restricted types
7. **Deleting an object with dynamic fields** renders those fields inaccessible — use Table/Bag with length checks to prevent this

## References

- [IOTA Dynamic Fields docs](https://docs.iota.org/developer/iota-101/objects/dynamic-fields/)
- [Tables and Bags](https://docs.iota.org/developer/iota-101/objects/dynamic-fields/tables-bags)
- [Object Model](https://docs.iota.org/developer/iota-101/objects/)
- [Transfer module](https://docs.iota.org/developer/references/framework/iota/transfer)
- [← Prev: Advanced Patterns](move-advanced.md) | [→ Next: DeFi in Move](move-defi.md)
