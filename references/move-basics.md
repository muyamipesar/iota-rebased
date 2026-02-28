# Move Language Basics for IOTA

## What is Move?

Move is a resource-oriented programming language designed for safe digital asset management. On IOTA, you write Move smart contracts that define objects, their behaviors, and access rules.

IOTA uses the **Move 2024 edition** with modern syntax features.

## Package Structure

```
my_package/
├── Move.toml          # Package manifest
├── sources/           # Move source files
│   └── my_module.move
└── tests/             # Test files
    └── my_module_tests.move
```

### Move.toml

```toml
[package]
name = "my_package"
edition = "2024"

[dependencies]
Iota = { git = "https://github.com/iotaledger/iota.git", subdir = "crates/iota-framework/packages/iota-framework", rev = "mainnet" }

[addresses]
my_package = "0x0"
```

## Basic Module

```move
module my_package::greeting {
    use std::string::String;

    /// A greeting object owned by someone
    public struct Greeting has key, store {
        id: UID,
        message: String,
    }

    /// Create a new greeting
    public fun create(message: String, ctx: &mut TxContext) {
        let greeting = Greeting {
            id: object::new(ctx),
            message,
        };
        transfer::transfer(greeting, ctx.sender());
    }

    /// Read the message
    public fun message(greeting: &Greeting): &String {
        &greeting.message
    }

    /// Update the message (only owner can call, since Greeting is owned)
    public fun update(greeting: &mut Greeting, new_message: String) {
        greeting.message = new_message;
    }

    /// Delete the greeting
    public fun destroy(greeting: Greeting) {
        let Greeting { id, message: _ } = greeting;
        id.delete();
    }
}
```

## Key Concepts

### Objects

Any struct with `key` ability and a `UID` field is an object:

```move
public struct Coin has key, store {
    id: UID,
    value: u64,
}
```

### Ownership & Transfer

```move
// Transfer to a specific address
transfer::transfer(obj, recipient);

// Make object shared (anyone can use)
transfer::share_object(obj);

// Make object immutable
transfer::freeze_object(obj);
```

**Best practice**: Return objects from functions instead of self-transferring. This enables composability.

### Entry Functions

Functions callable directly from transactions:

```move
entry fun mint_nft(name: String, ctx: &mut TxContext) {
    let nft = NFT { id: object::new(ctx), name };
    transfer::transfer(nft, ctx.sender());
}
```

### Coins and Balance

```move
use iota::coin::{Self, Coin};
use iota::balance::Balance;

// Accept exact payment
public fun buy(payment: Coin<IOTA>, ctx: &mut TxContext) {
    // Don't use &mut Coin + amount — use exact Coin for safety
    let value = payment.value();
    assert!(value >= 1000, EInsufficientPayment);
    // ...
}
```

## Move 2024 Features

### Method Syntax

```move
// Old: vector::push_back(&mut v, 42);
v.push_back(42);

// Old: coin::value(&c)
c.value();
```

### Enums

```move
public enum Status {
    Active,
    Paused,
    Completed(u64),  // with data
}

public fun is_active(s: &Status): bool {
    match (s) {
        Status::Active => true,
        _ => false,
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
let val = v[0];        // vector::borrow(&v, 0)
*&mut v[1] = 42;       // vector::borrow_mut(&mut v, 1)
```

### public(package)

Replaces `public(friend)` — function visible only within the same package:

```move
public(package) fun internal_helper() { /* ... */ }
```

## Auto-imported in Every Module

```move
use std::vector;
use std::option::{Self, Option};
use iota::object::{Self, ID, UID};
use iota::transfer;
use iota::tx_context::TxContext;
```

## Collections

| Type | Backing | Use When |
|------|---------|----------|
| `vector` | In-object | Known max ≤ 1000 items |
| `VecSet`, `VecMap` | In-object | Small key-value or set data |
| `Table`, `Bag` | Dynamic fields | Large/unbounded collections |
| `ObjectTable`, `ObjectBag` | Dynamic fields | Collections of objects |

**Max object size: 250KB** — use dynamic field collections for anything that grows.

## Testing

```move
#[test]
fun test_greeting() {
    let mut ctx = tx_context::dummy();
    let greeting = Greeting {
        id: object::new(&mut ctx),
        message: string::utf8(b"Hello"),
    };
    assert!(greeting.message() == &string::utf8(b"Hello"));
    destroy(greeting);
}
```

Use `iota::test_scenario` for multi-transaction tests simulating different senders.

```bash
iota move test              # Run tests
iota move test --coverage   # With coverage
iota move coverage source --module my_module  # See uncovered lines
```

## References

- [Move 2024 features](https://docs.iota.org/developer/advanced/introducing-move-2024)
- [IOTA 101 - Move overview](https://docs.iota.org/developer/iota-101/move-overview/)
- [Move coding conventions](https://move-language.github.io/move/coding-conventions.html)
- [From Solidity to Move](https://docs.iota.org/developer/evm-to-move/)

## Next Steps

Ready for more? Continue to [move-advanced.md](move-advanced.md) for design patterns (Witness, Hot Potato, Capability, OTW), then [move-objects-fields.md](move-objects-fields.md) for the object model deep dive.
