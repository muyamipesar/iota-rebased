# Security Best Practices for Move on IOTA

## Table of Contents

- [Overview](#overview)
- [1. Why Move Is Inherently Safer](#1-why-move-is-inherently-safer)
- [2. Access Control Patterns](#2-access-control-patterns)
- [3. Input Validation](#3-input-validation)
- [4. Arithmetic Safety](#4-arithmetic-safety)
- [5. Object Safety](#5-object-safety)
- [6. Reentrancy — Why It Doesn't Exist in Move](#6-reentrancy--why-it-doesnt-exist-in-move)
- [7. Visibility and Encapsulation](#7-visibility-and-encapsulation)
- [8. Upgradeability Risks](#8-upgradeability-risks)
- [9. Common Mistakes and How to Avoid Them](#9-common-mistakes-and-how-to-avoid-them)
- [10. Security Checklist](#10-security-checklist)
- [References](#references)

## Overview

Move was designed from the ground up for asset safety. Its type system prevents entire classes of vulnerabilities common in Solidity/EVM. However, logic bugs and access control errors are still possible.

## 1. Why Move Is Inherently Safer

| Vulnerability | Solidity/EVM | Move on IOTA |
|---------------|-------------|--------------|
| Reentrancy | Major risk | **Impossible** — no dynamic dispatch |
| Integer overflow | Possible (pre-0.8) | **Impossible** — runtime abort on overflow |
| Uninitialized storage | Possible | **Impossible** — all values initialized |
| Dangling references | Possible | **Impossible** — borrow checker enforced |
| Double-spend / duplication | Possible via bugs | **Impossible** — resources are linear types |
| Access control bypass | Common | Prevented by **capability pattern** |
| Storage collision | Possible (proxy patterns) | **Impossible** — typed dynamic fields |

**What Move does NOT protect against:**
- Logic bugs (wrong formulas, wrong conditions)
- Access control mistakes (forgetting to check a capability)
- Economic exploits (oracle manipulation, flash loan attacks)
- Denial of service (unbounded loops, gas exhaustion)

## 2. Access Control Patterns

### Always Use Capabilities for Admin Functions

```move
// ✅ Correct — requires capability proof
public fun admin_action(cap: &AdminCap, target: &mut Pool) {
    // Only holder of AdminCap can call this
    target.paused = true;
}

// ❌ Wrong — anyone can call this
public fun admin_action(target: &mut Pool, ctx: &TxContext) {
    assert!(tx_context::sender(ctx) == @admin_address, ENotAdmin);
    target.paused = true;
}
```

**Why capabilities > address checks:**
- Capabilities are objects — they can be transferred, revoked, multi-sig'd
- Address checks are brittle — the admin address is hardcoded
- Capabilities compose with PTBs and other patterns

### Separate Read and Write Capabilities

```move
public struct ReadCap has key, store { id: UID }
public struct WriteCap has key, store { id: UID }
public struct AdminCap has key, store { id: UID }

/// Read-only access
public fun get_data(_: &ReadCap, pool: &Pool): u64 { pool.value }

/// Write access
public fun update_data(_: &WriteCap, pool: &mut Pool, val: u64) { pool.value = val; }

/// Admin-only: can create new capabilities
public fun grant_read_access(_: &AdminCap, ctx: &mut TxContext): ReadCap {
    ReadCap { id: object::new(ctx) }
}
```

## 3. Input Validation

### Validate All External Inputs

```move
const EZeroAmount: u64 = 0;
const EExceedsMax: u64 = 1;
const EInvalidRecipient: u64 = 2;

const MAX_AMOUNT: u64 = 1_000_000_000;

public fun deposit(pool: &mut Pool, coin: Coin<IOTA>) {
    let amount = coin::value(&coin);

    // Validate amount
    assert!(amount > 0, EZeroAmount);
    assert!(amount <= MAX_AMOUNT, EExceedsMax);

    balance::join(&mut pool.funds, coin::into_balance(coin));
}
```

### Use Named Error Constants

```move
// ✅ Good — descriptive error codes
const ENotAuthorized: u64 = 0;
const EInsufficientBalance: u64 = 1;
const EPoolPaused: u64 = 2;

// ❌ Bad — magic numbers
assert!(balance > amount, 42);
```

### Accept Exact Coins Instead of Mutable References

```move
// ✅ Safe — caller knows exactly what they pay
public fun buy(payment: Coin<IOTA>): Item {
    assert!(coin::value(&payment) == PRICE, EWrongPrice);
    transfer::public_transfer(payment, @treasury);
    // ...
}

// ❌ Risky — caller must trust function to take correct amount
public fun buy(payment: &mut Coin<IOTA>, ctx: &mut TxContext): Item {
    let paid = coin::split(payment, PRICE, ctx);
    // ...
}
```

## 4. Arithmetic Safety

Move aborts on integer overflow/underflow at runtime, so you're safe from silent wrapping. But you still need to handle division and ordering:

```move
// Division by zero aborts at runtime — but give a better error:
public fun calculate_share(total: u64, parts: u64): u64 {
    assert!(parts > 0, EDivisionByZero);
    total / parts
}

// Subtraction underflow — check before subtracting:
public fun safe_subtract(a: u64, b: u64): u64 {
    assert!(a >= b, EUnderflow);
    a - b
}

// Multiplication overflow risk with large numbers — use u128 intermediaries:
public fun mul_div(a: u64, b: u64, denominator: u64): u64 {
    assert!(denominator > 0, EDivisionByZero);
    let result = ((a as u128) * (b as u128)) / (denominator as u128);
    assert!(result <= (18446744073709551615u128), EOverflow); // MAX_U64
    (result as u64)
}
```

## 5. Object Safety

### Don't Delete Objects with Dynamic Fields

```move
// ❌ Dangerous — dynamic fields become permanently inaccessible
public fun destroy_registry(registry: Registry) {
    let Registry { id } = registry;
    object::delete(id);
    // All dynamic fields on this object are now orphaned!
}

// ✅ Safe — use Table/Bag with length checks
public fun destroy_registry(registry: Registry) {
    let Registry { id, items } = registry;
    assert!(table::is_empty(&items), ENotEmpty);
    table::destroy_empty(items);
    object::delete(id);
}
```

### Be Careful with Shared Object State

```move
// ❌ Dangerous — time-of-check vs time-of-use on shared objects
// Between checking and acting, another tx could change state
public fun withdraw_if_enough(pool: &mut Pool, amount: u64, ctx: &mut TxContext): Coin<IOTA> {
    // This is fine because the borrow is atomic within one PTB command,
    // but be careful with multi-step logic across PTB commands
    assert!(balance::value(&pool.funds) >= amount, EInsufficient);
    let bal = balance::split(&mut pool.funds, amount);
    coin::from_balance(bal, ctx)
}
```

## 6. Reentrancy — Why It Doesn't Exist in Move

In Solidity, reentrancy occurs when a contract calls an external contract, which calls back into the original contract before the first call completes.

**Move makes this impossible because:**

1. **No dynamic dispatch** — you can't call arbitrary code. All function calls are statically resolved at compile time.
2. **No callbacks** — Move functions can't receive function pointers or closures (macros are inlined at compile time).
3. **Borrow checker** — if you have a `&mut` reference to an object, no one else can access it simultaneously.
4. **Linear resources** — objects can't be duplicated or silently dropped.

**Bottom line:** You don't need reentrancy guards in Move. The language prevents it structurally.

## 7. Visibility and Encapsulation

### Use the Minimum Necessary Visibility

```move
// ✅ Internal helper — not exposed
fun internal_calculate(x: u64): u64 { x * 2 }

// ✅ Package-only — usable by other modules in your package
public(package) fun package_helper(x: u64): u64 { x + 1 }

// ✅ Public — permanent API, can never be removed
public fun stable_api(x: u64): u64 { x }
```

**Rules:**
- `fun` (private) — default, only this module
- `public(package)` — any module in the same package
- `public` — anyone, forever (cannot be removed in upgrades)

### Don't Add `store` Unless Needed

Adding `store` to a type makes it transferable by anyone. Omit it for controlled transfers:

```move
// Anyone can transfer this (has `store`)
public struct TransferableNFT has key, store { id: UID }

// Only this module can transfer this (no `store`)
public struct ControlledNFT has key { id: UID }
```

## 8. Upgradeability Risks

When you publish a package, you get an `UpgradeCap`. Upgrades have restrictions:

**What you CAN do in an upgrade:**
- Add new functions
- Add new modules
- Add new struct types
- Change function implementations (private and `public(package)`)

**What you CANNOT do:**
- Remove or change `public` function signatures
- Remove struct types
- Add fields to existing structs (use dynamic fields instead)
- Add new abilities to existing types
- Change module names

### Protect the UpgradeCap

```move
// Option 1: Make package immutable (no more upgrades)
public fun make_immutable(cap: UpgradeCap) {
    let UpgradeCap { id, .. } = cap;  // Destructure and delete
    object::delete(id);
}

// Option 2: Restrict upgrade policy
use iota::package;
public fun restrict_upgrades(cap: &mut UpgradeCap) {
    package::only_dep_upgrades(cap);  // Only dependency upgrades allowed
}
```

## 9. Common Mistakes and How to Avoid Them

### 1. Self-Transferring Instead of Returning

```move
// ❌ Breaks composability
public fun mint(ctx: &mut TxContext) {
    let nft = NFT { id: object::new(ctx) };
    transfer::transfer(nft, tx_context::sender(ctx));
}

// ✅ Let the caller decide
public fun mint(ctx: &mut TxContext): NFT {
    NFT { id: object::new(ctx) }
}
```

### 2. Unbounded Vectors

```move
// ❌ Can grow past 250KB object limit
public struct Registry has key {
    id: UID,
    entries: vector<Entry>,  // Will eventually hit 250KB
}

// ✅ Use Table for unbounded collections
public struct Registry has key {
    id: UID,
    entries: Table<u64, Entry>,
}
```

### 3. Forgetting to Verify OTW

```move
// ❌ Doesn't verify — any type with `drop` could be passed
public fun create_currency<T: drop>(witness: T, ctx: &mut TxContext) { ... }

// ✅ Verify it's a genuine OTW
use iota::types;
public fun create_currency<T: drop>(witness: T, ctx: &mut TxContext) {
    assert!(types::is_one_time_witness(&witness), ENotOTW);
    // ...
}
```

### 4. Not Handling All Coin Amounts

```move
// ❌ What if payment is more than PRICE? Extra coins are lost.
public fun buy(payment: Coin<IOTA>, ctx: &mut TxContext): Item {
    assert!(coin::value(&payment) >= PRICE, ETooLow);
    transfer::public_transfer(payment, @treasury);  // Overpayment goes to treasury
    // ...
}

// ✅ Exact payment or return change
public fun buy(mut payment: Coin<IOTA>, ctx: &mut TxContext): Item {
    assert!(coin::value(&payment) >= PRICE, ETooLow);
    let paid = coin::split(&mut payment, PRICE, ctx);
    transfer::public_transfer(paid, @treasury);
    transfer::public_transfer(payment, tx_context::sender(ctx));  // Return change
    // ...
}
```

## 10. Security Checklist

Before deploying to mainnet:

- [ ] **All admin functions gated by capability objects** (not address checks)
- [ ] **All inputs validated** — amounts > 0, within bounds, correct types
- [ ] **Named error constants** for every `assert!`
- [ ] **No unbounded vectors** in objects — use Table/Bag for collections
- [ ] **Functions return objects** instead of self-transferring (composability)
- [ ] **Minimum visibility** — `fun` by default, `public` only when necessary
- [ ] **`store` added intentionally** — only when you want unrestricted transfers
- [ ] **100% test coverage** (or as close as possible)
- [ ] **Test failure paths** — expected_failure tests for every error code
- [ ] **UpgradeCap secured** — stored safely or policy restricted
- [ ] **No objects deleted with active dynamic fields** — use Table/Bag
- [ ] **Coin payments accept exact amounts** or return change
- [ ] **OTW verified** with `types::is_one_time_witness` where applicable
- [ ] **Upgrade plan considered** — can't change public signatures or struct layouts

## References

- [IOTA Dev Cheat Sheet — Move section](https://docs.iota.org/developer/dev-cheat-sheet)
- [Move Coding Conventions](https://move-language.github.io/move/coding-conventions.html)
- [Package Upgrades](https://docs.iota.org/developer/iota-101/move-overview/package-upgrades/)
- [← Prev: Testing](move-testing.md) | [→ Next: Gas Optimization](move-gas.md)
