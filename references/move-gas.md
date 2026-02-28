# Gas Optimization on IOTA

## Table of Contents

- [Overview](#overview)
- [1. How Gas Works on IOTA](#1-how-gas-works-on-iota)
- [2. Gas Cost Components](#2-gas-cost-components)
- [3. Owned vs Shared Objects](#3-owned-vs-shared-objects)
- [4. Object Size Optimization](#4-object-size-optimization)
- [5. Collection Choice](#5-collection-choice)
- [6. Programmable Transaction Blocks](#6-programmable-transaction-blocks)
- [7. Code-Level Tips](#7-code-level-tips)
- [8. When NOT to Optimize](#8-when-not-to-optimize)
- [9. Measuring Gas Cost](#9-measuring-gas-cost)
- [References](#references)

## Overview

IOTA Rebased has **very low gas fees** (~0.005 IOTA per simple transaction). Gas is priced in NANOS (1 IOTA = 1,000,000,000 NANOS). All gas fees are **burned**, reducing total supply over time.

> **Key insight from the IOTA cheat sheet:** "Don't micro-optimize gas usage. IOTA computation costs are rounded up to the closest bucket, so only very drastic changes will make a difference. If your transaction is already in the lowest cost bucket, it can't get any cheaper."

That said, understanding gas helps you design better systems — especially for high-throughput applications.

## 1. How Gas Works on IOTA

Every transaction specifies:
- **Gas budget**: Maximum gas units the tx can consume (aborts if exceeded)
- **Gas price**: NANOS per gas unit (must be ≥ reference gas price set by validators)
- **Gas payment object**: A `Coin<IOTA>` owned by the sender (or sponsor)

```bash
# Set gas budget on CLI commands
iota client call --gas-budget 10000000 ...

# Check reference gas price
iota client gas-price
```

## 2. Gas Cost Components

| Component | What It Costs | Typical Range |
|-----------|--------------|---------------|
| **Computation** | CPU cycles for execution | Bucketed (see below) |
| **Storage** | New object creation / growth | Per byte stored |
| **Storage rebate** | Returned when objects are deleted | Offsets deletion cost |

### Computation Buckets

Computation costs are rounded to buckets. Small optimizations within a bucket have **zero effect**:

```
Bucket 1: 1,000,000 NANOS  (simple transfers, reads)
Bucket 2: 2,500,000 NANOS  (moderate computation)
Bucket 3: 5,000,000 NANOS  (complex logic)
...etc
```

Only reducing computation enough to drop to a lower bucket saves gas.

## 3. Owned vs Shared Objects

**This is the single biggest performance decision in your architecture.**

| | Owned Objects | Shared Objects |
|---|--------------|----------------|
| Consensus needed? | No | Yes |
| Parallel execution? | Yes | No (sequentialized) |
| Gas cost | Lower | Higher (consensus overhead) |
| Use when | User-specific assets | Multi-user shared state |

### Design for Owned Objects Where Possible

```move
// ❌ Expensive — shared counter requires consensus for every increment
public struct GlobalCounter has key {
    id: UID,
    count: u64,
}

// ✅ Cheaper — each user has their own counter (parallel execution)
public struct UserCounter has key {
    id: UID,
    owner: address,
    count: u64,
}
```

### Split Shared State When Possible

```move
// ❌ One shared object for everything — all trades sequentialized
public struct Exchange has key {
    id: UID,
    pool_a: Balance<TokenA>,
    pool_b: Balance<TokenB>,
    pool_c: Balance<TokenC>,
}

// ✅ Separate pool per pair — different pairs can trade in parallel
public struct Pool<phantom A, phantom B> has key {
    id: UID,
    reserve_a: Balance<A>,
    reserve_b: Balance<B>,
}
```

## 4. Object Size Optimization

Objects have a **250 KB maximum**. Larger objects also cost more storage gas.

```move
// ❌ Stores data inline — grows with usage
public struct Registry has key {
    id: UID,
    entries: vector<Entry>,  // Each entry increases object size
}

// ✅ Uses dynamic fields — base object stays small
public struct Registry has key {
    id: UID,
    count: u64,  // Just metadata
    // Entries stored as dynamic fields — only loaded when accessed
}
```

### Delete Objects to Get Storage Rebates

When you delete an object, you get a **storage rebate** — a refund of the storage cost originally paid:

```move
public fun cleanup(item: Item) {
    let Item { id, data: _ } = item;
    object::delete(id);
    // Storage rebate returned to transaction sender
}
```

## 5. Collection Choice

| Collection | Cost Profile | Best For |
|-----------|-------------|----------|
| `vector<T>` | O(n) iteration, all in object | Small known-size collections (≤1000) |
| `VecMap<K, V>` | O(n) lookup, all in object | Small maps (≤1000) |
| `Table<K, V>` | O(1) lookup, dynamic fields | Large/unbounded collections |
| `Bag` | O(1) lookup, heterogeneous | Mixed-type large collections |

**Rule:** If the collection can grow beyond 1000 items or is user-contributed, use `Table`/`Bag`.

## 6. Programmable Transaction Blocks (PTBs)

PTBs let you **batch multiple operations** in a single transaction. This is dramatically cheaper than multiple separate transactions:

```bash
# Single PTB that does 3 things:
# 1. Split a coin
# 2. Transfer part to Alice
# 3. Transfer part to Bob
iota client ptb \
    --split-coins @gas_coin [1000, 2000] \
    --transfer-objects [@result.0] @alice \
    --transfer-objects [@result.1] @bob \
    --gas-budget 10000000
```

In TypeScript SDK:

```typescript
import { Transaction } from '@iota/iota-sdk/transactions';

const tx = new Transaction();

// Batch operations in one tx
const [coin1, coin2] = tx.splitCoins(tx.gas, [1000, 2000]);
tx.transferObjects([coin1], alice);
tx.transferObjects([coin2], bob);

// One transaction, one gas payment
await client.signAndExecuteTransaction({ signer: keypair, transaction: tx });
```

**PTB benefits:**
- One gas payment for multiple operations
- Intermediate results can be passed between commands (composability)
- Atomic — all operations succeed or all fail

## 7. Code-Level Tips

### Avoid Unnecessary Object Creation

```move
// ❌ Creates intermediate coin object
public fun transfer_amount(from: &mut Coin<IOTA>, to: address, amount: u64, ctx: &mut TxContext) {
    let coin = coin::split(from, amount, ctx);  // New object created
    transfer::public_transfer(coin, to);
}

// ✅ Use Balance for internal accounting (no object overhead)
public fun internal_transfer(from: &mut Balance<IOTA>, to: &mut Balance<IOTA>, amount: u64) {
    let transferred = balance::split(from, amount);
    balance::join(to, transferred);
}
```

### Minimize Shared Object Mutations

Each mutation to a shared object requires consensus. Batch writes when possible:

```move
// ❌ Multiple calls to shared object
public fun update_score(board: &mut Leaderboard, player: address, score: u64) { ... }
// Called 100 times = 100 consensus rounds

// ✅ Batch update in one call
public fun update_scores(board: &mut Leaderboard, players: vector<address>, scores: vector<u64>) {
    let mut i = 0;
    while (i < players.length()) {
        // Update all at once — one consensus round
        set_score_internal(board, players[i], scores[i]);
        i = i + 1;
    };
}
```

### Use Events Instead of On-Chain Storage for Logs

```move
use iota::event;

public struct TradeExecuted has copy, drop {
    trader: address,
    amount_in: u64,
    amount_out: u64,
}

public fun execute_trade(...) {
    // ... trade logic ...

    // Emit event — cheaper than storing history on-chain
    event::emit(TradeExecuted { trader: tx_context::sender(ctx), amount_in, amount_out });
}
```

Events are indexed off-chain and queryable but don't cost per-byte storage fees forever.

## 8. When NOT to Optimize

Per the IOTA cheat sheet:

1. **If you're in the lowest computation bucket** — it can't get cheaper
2. **If your app does < 1000 tx/day** — gas savings are negligible
3. **If it reduces code clarity** — readability > micro-optimization
4. **If it compromises security** — never skip validation to save gas

**Focus optimization energy on:**
- Choosing owned vs shared objects (architecture-level)
- Using PTBs to batch operations (transaction-level)
- Proper collection types (data structure-level)

## 9. Measuring Gas Cost

### Dry Run

```bash
iota client call \
    --package 0x<PKG> \
    --module my_module \
    --function my_function \
    --args ... \
    --dry-run
```

Shows gas cost without executing.

### From Transaction Results

After executing a transaction, check the Gas Cost Summary:

```
Gas Cost Summary:
    Storage Cost: 14424800 NANOS
    Computation Cost: 1000000 NANOS
    Storage Rebate: 980400 NANOS
    Non-refundable Storage Fee: 0 NANOS
```

### SDK

```typescript
const result = await client.signAndExecuteTransaction({
    signer: keypair,
    transaction: tx,
    options: { showEffects: true },
});

console.log(result.effects.gasUsed);
// { computationCost, storageCost, storageRebate, nonRefundableStorageFee }
```

## References

- [IOTA Dev Cheat Sheet](https://docs.iota.org/developer/dev-cheat-sheet)
- [Gas in IOTA](https://docs.iota.org/developer/iota-101/transactions/gas)
- [PTB docs](https://docs.iota.org/developer/iota-101/transactions/ptb/programmable-transaction-blocks)
- [← Prev: Security](move-security.md) | [→ Next: IOTA vs Sui](move-vs-sui.md)
