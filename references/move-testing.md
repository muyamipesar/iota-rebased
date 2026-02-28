# Testing Move Smart Contracts on IOTA

## Table of Contents

- [Overview](#overview)
- [1. Unit Tests](#1-unit-tests)
- [2. Test Attributes](#2-test-attributes)
- [3. Test Scenario (Multi-Tx Testing)](#3-test-scenario-multi-tx-testing)
- [4. Test Utilities](#4-test-utilities)
- [5. Testing Coin and Token Operations](#5-testing-coin-and-token-operations)
- [6. Testing Shared Objects](#6-testing-shared-objects)
- [7. Code Coverage](#7-code-coverage)
- [8. Integration Testing with CLI](#8-integration-testing-with-cli)
- [9. Best Practices](#9-best-practices)
- [References](#references)

## Overview

IOTA uses the Move testing framework with IOTA-specific test utilities. Tests run locally — no network needed.

```bash
iota move test                    # Run all tests
iota move test -f test_name       # Run specific test
iota move test --coverage         # Run with coverage tracking
iota move coverage source --module my_module  # Show uncovered lines
```

## 1. Unit Tests

Test functions are annotated with `#[test]` and placed in the same file or in `tests/`:

```move
module examples::math {
    const EOverflow: u64 = 0;

    public fun safe_add(a: u64, b: u64): u64 {
        let result = a + b;
        assert!(result >= a, EOverflow);
        result
    }

    #[test]
    fun test_safe_add() {
        assert!(safe_add(1, 2) == 3, 0);
        assert!(safe_add(0, 0) == 0, 0);
        assert!(safe_add(100, 200) == 300, 0);
    }

    #[test]
    #[expected_failure(abort_code = EOverflow)]
    fun test_safe_add_overflow() {
        // This should abort with EOverflow
        safe_add(18446744073709551615, 1);
    }
}
```

## 2. Test Attributes

| Attribute | Purpose |
|-----------|---------|
| `#[test]` | Mark function as a test |
| `#[test_only]` | Mark function/module only compiled for tests |
| `#[expected_failure]` | Test expects an abort |
| `#[expected_failure(abort_code = N)]` | Expects abort with specific code |
| `#[expected_failure(abort_code = module::ERROR)]` | Expects abort from specific module |

### Test-Only Code

```move
module examples::my_module {
    public struct Secret has key {
        id: UID,
        value: u64,
    }

    public fun create(value: u64, ctx: &mut TxContext): Secret {
        Secret { id: object::new(ctx), value }
    }

    /// Only available in tests — allows destroying for assertions.
    #[test_only]
    public fun destroy_for_testing(secret: Secret): u64 {
        let Secret { id, value } = secret;
        object::delete(id);
        value
    }
}
```

## 3. Test Scenario (Multi-Tx Testing)

`iota::test_scenario` simulates multi-transaction, multi-sender flows:

```move
#[test_only]
module examples::my_module_tests {
    use examples::my_module::{Self, AdminCap, Item};
    use iota::test_scenario;

    #[test]
    fun test_admin_flow() {
        let admin = @0xAD;
        let user = @0xB0B;

        // === Tx 1: Deploy (init runs automatically) ===
        let mut scenario = test_scenario::begin(admin);
        {
            // init() is called automatically for the module
            my_module::init_for_testing(scenario.ctx());
        };

        // === Tx 2: Admin creates an item ===
        scenario.next_tx(admin);
        {
            let cap = scenario.take_from_sender<AdminCap>();
            let item = my_module::create_item(
                &cap,
                b"Sword".to_string(),
                scenario.ctx(),
            );
            transfer::public_transfer(item, user);
            scenario.return_to_sender(cap);
        };

        // === Tx 3: User verifies they received it ===
        scenario.next_tx(user);
        {
            let item = scenario.take_from_sender<Item>();
            // Assert properties...
            scenario.return_to_sender(item);
        };

        scenario.end();
    }
}
```

### Key Test Scenario Functions

```move
use iota::test_scenario::{Self, Scenario};

// Start a new scenario as a specific sender
let mut scenario = test_scenario::begin(@0xADMIN);

// Advance to next transaction with a new sender
scenario.next_tx(@0xUSER);

// Take an object owned by the current sender
let obj = scenario.take_from_sender<MyType>();

// Return object to the sender (must do this or transfer it)
scenario.return_to_sender(obj);

// Take a shared object
let shared = scenario.take_shared<SharedPool>();
test_scenario::return_shared(shared);

// Take an immutable object
let imm = scenario.take_immutable<Config>();
test_scenario::return_immutable(imm);

// Get the TxContext
let ctx = scenario.ctx();

// Check how many objects a sender has
let count = scenario.ids_for_sender<MyType>().length();

// End the scenario (required — cleans up)
scenario.end();
```

## 4. Test Utilities

`iota::test_utils` provides helpers:

```move
use iota::test_utils;

#[test]
fun test_with_utils() {
    // Better assertion with error message
    test_utils::assert_eq(actual, expected);

    // Debug print (shows in test output)
    test_utils::print(b"Debug message");

    // Destroy any object in tests (useful for cleanup)
    test_utils::destroy(some_object);
}
```

### Creating Test Coins

```move
use iota::coin;
use iota::iota::IOTA;

#[test]
fun test_with_coins() {
    let mut scenario = test_scenario::begin(@0x1);
    let ctx = scenario.ctx();

    // Create a test coin with specific value
    let coin = coin::mint_for_testing<IOTA>(1000, ctx);
    assert!(coin::value(&coin) == 1000, 0);

    // Clean up
    test_utils::destroy(coin);
    scenario.end();
}
```

## 5. Testing Coin and Token Operations

```move
#[test_only]
module examples::token_tests {
    use examples::my_token::{Self, MY_TOKEN};
    use iota::coin::{Self, TreasuryCap};
    use iota::test_scenario;

    #[test]
    fun test_mint_and_transfer() {
        let admin = @0xAD;
        let recipient = @0xBB;

        let mut scenario = test_scenario::begin(admin);

        // Tx 1: init creates TreasuryCap
        {
            my_token::init_for_testing(scenario.ctx());
        };

        // Tx 2: Mint tokens
        scenario.next_tx(admin);
        {
            let mut treasury = scenario.take_from_sender<TreasuryCap<MY_TOKEN>>();
            my_token::mint(&mut treasury, 1000, recipient, scenario.ctx());
            scenario.return_to_sender(treasury);
        };

        // Tx 3: Recipient checks balance
        scenario.next_tx(recipient);
        {
            let coin = scenario.take_from_sender<Coin<MY_TOKEN>>();
            assert!(coin::value(&coin) == 1000, 0);
            scenario.return_to_sender(coin);
        };

        scenario.end();
    }
}
```

## 6. Testing Shared Objects

```move
#[test]
fun test_shared_pool() {
    let creator = @0x1;
    let user = @0x2;

    let mut scenario = test_scenario::begin(creator);

    // Tx 1: Create shared pool
    {
        let pool = Pool {
            id: object::new(scenario.ctx()),
            balance: 0,
        };
        transfer::share_object(pool);
    };

    // Tx 2: User interacts with shared pool
    scenario.next_tx(user);
    {
        let mut pool = scenario.take_shared<Pool>();
        pool.balance = pool.balance + 100;
        test_scenario::return_shared(pool);
    };

    // Tx 3: Verify
    scenario.next_tx(creator);
    {
        let pool = scenario.take_shared<Pool>();
        assert!(pool.balance == 100, 0);
        test_scenario::return_shared(pool);
    };

    scenario.end();
}
```

## 7. Code Coverage

```bash
# Run tests with coverage tracking
iota move test --coverage

# View coverage summary
iota move coverage summary

# View line-by-line coverage for a module (uncovered lines in red)
iota move coverage source --module my_module
```

**Target 100% coverage** when feasible. The coverage tool shows exactly which lines are untested.

## 8. Integration Testing with CLI

For end-to-end testing against a local network:

```bash
# Start a local network
RUST_LOG="off,iota_node=info" cargo run --bin iota start --force-regenesis --with-faucet

# In another terminal:
iota client switch --env localnet

# Publish
iota client publish --gas-budget 100000000

# Call a function
iota client call \
    --package 0x<PACKAGE_ID> \
    --module my_module \
    --function create_item \
    --args 0x<ADMIN_CAP_ID> '"My Item"' \
    --gas-budget 10000000

# Query an object
iota client object 0x<OBJECT_ID>
```

### Dry Run (Simulate Without Executing)

```bash
iota client call \
    --package 0x<PACKAGE_ID> \
    --module my_module \
    --function some_function \
    --args ... \
    --dry-run
```

## 9. Best Practices

1. **Test every public function** — unit tests for logic, scenario tests for flows
2. **Test error paths** — use `#[expected_failure(abort_code = ...)]`
3. **Use `#[test_only]`** for test helpers — keeps production code clean
4. **Always call `scenario.end()`** — prevents resource leaks in tests
5. **Return or destroy all objects** — every object taken must be returned, transferred, or destroyed
6. **Test with realistic values** — use proper coin amounts, not just 1
7. **Push coverage to 100%** when feasible — use `iota move test --coverage`
8. **Test multi-sender flows** — use `scenario.next_tx(different_address)` to simulate real interactions
9. **Create `init_for_testing`** functions — wrap `init` logic for test access:

```move
#[test_only]
public fun init_for_testing(ctx: &mut TxContext) {
    init(MY_TOKEN {}, ctx)
}
```

## References

- [iota::test_scenario source](https://github.com/iotaledger/iota/blob/develop/crates/iota-framework/packages/iota-framework/sources/test/test_scenario.move)
- [iota::test_utils source](https://github.com/iotaledger/iota/blob/develop/crates/iota-framework/packages/iota-framework/sources/test/test_utils.move)
- [IOTA Build & Test docs](https://docs.iota.org/developer/getting-started/build-test)
- [← Prev: DeFi Patterns](move-defi.md) | [→ Next: Security](move-security.md)
