# Smart Contracts on IOTA Rebased

## Creating a Move Smart Contract

### 1. Create Package

```bash
iota move new my_defi_app
cd my_defi_app
```

### 2. Write Your Contract

`sources/my_defi_app.move`:

```move
module my_defi_app::vault {
    use iota::coin::{Self, Coin};
    use iota::balance::{Self, Balance};
    use iota::iota::IOTA;

    /// A shared vault that holds IOTA tokens
    public struct Vault has key {
        id: UID,
        balance: Balance<IOTA>,
    }

    /// Create a new shared vault
    fun init(ctx: &mut TxContext) {
        let vault = Vault {
            id: object::new(ctx),
            balance: balance::zero(),
        };
        transfer::share_object(vault);
    }

    /// Deposit IOTA into the vault
    public fun deposit(vault: &mut Vault, coin: Coin<IOTA>) {
        vault.balance.join(coin.into_balance());
    }

    /// Withdraw IOTA from the vault (only package can call)
    public(package) fun withdraw(
        vault: &mut Vault,
        amount: u64,
        ctx: &mut TxContext,
    ): Coin<IOTA> {
        let withdrawn = vault.balance.split(amount);
        coin::from_balance(withdrawn, ctx)
    }

    /// Check balance
    public fun balance(vault: &Vault): u64 {
        vault.balance.value()
    }
}
```

### 3. Build

```bash
iota move build
```

### 4. Test

```move
#[test_only]
module my_defi_app::vault_tests {
    use my_defi_app::vault;
    use iota::test_scenario;
    use iota::coin;
    use iota::iota::IOTA;

    #[test]
    fun test_deposit_and_balance() {
        let admin = @0xAD;
        let mut scenario = test_scenario::begin(admin);

        // Init creates the shared vault
        {
            vault::init(scenario.ctx());
        };

        // Deposit
        scenario.next_tx(admin);
        {
            let mut v = scenario.take_shared<vault::Vault>();
            let payment = coin::mint_for_testing<IOTA>(1000, scenario.ctx());
            vault::deposit(&mut v, payment);
            assert!(vault::balance(&v) == 1000);
            test_scenario::return_shared(v);
        };

        scenario.end();
    }
}
```

```bash
iota move test
iota move test --coverage
```

### 5. Deploy to Testnet

```bash
# Make sure you're on testnet
iota client switch --env testnet

# Get test tokens if needed
# curl -X POST https://faucet.testnet.iota.cafe/gas -H 'Content-Type: application/json' -d '{"FixedAmountRequest":{"recipient":"YOUR_ADDRESS"}}'

# Publish
iota client publish --gas-budget 100000000
```

The output gives you the **Package ID** — save it for interactions.

### 6. Interact with Your Contract

```bash
# Call a function
iota client call \
    --package 0xYOUR_PACKAGE_ID \
    --module vault \
    --function deposit \
    --args 0xVAULT_OBJECT_ID 0xCOIN_OBJECT_ID \
    --gas-budget 10000000
```

## Deploy to Mainnet

```bash
iota client new-env --alias mainnet --rpc https://api.mainnet.iota.cafe
iota client switch --env mainnet
iota client publish --gas-budget 100000000
```

## Package Upgrades

Published packages are **immutable** but can be upgraded:

- You **cannot** delete or change public function signatures
- You **cannot** delete struct types or add new abilities to existing structs
- You **can** add new functions and new struct types
- Use `public(package)` for functions you may want to change later

```bash
iota client upgrade --upgrade-capability 0xCAP_ID --gas-budget 100000000
```

## Init Functions

The `init` function runs once at publish time. Use it to create shared objects, set up admin caps, etc.

```move
fun init(ctx: &mut TxContext) {
    // Runs only on publish
    let admin_cap = AdminCap { id: object::new(ctx) };
    transfer::transfer(admin_cap, ctx.sender());
}
```

## Patterns

### Capability Pattern (Access Control)

```move
public struct AdminCap has key, store { id: UID }

public fun admin_only_action(_cap: &AdminCap, /* ... */) {
    // Only someone with AdminCap can call this
}
```

### One-Time Witness (OTW)

```move
public struct MY_TOKEN has drop {}

fun init(witness: MY_TOKEN, ctx: &mut TxContext) {
    // witness proves this is the first and only init call
}
```

### Display Standard

```move
use iota::display;

fun init(otw: MY_NFT, ctx: &mut TxContext) {
    let publisher = package::claim(otw, ctx);
    let mut d = display::new_with_fields<MyNFT>(
        &publisher,
        vector[string::utf8(b"name"), string::utf8(b"image_url")],
        vector[string::utf8(b"{name}"), string::utf8(b"{image_url}")],
        ctx,
    );
    d.update_version();
    transfer::public_transfer(publisher, ctx.sender());
    transfer::public_transfer(d, ctx.sender());
}
```

## Best Practices

1. **Use exact `Coin<T>` params** instead of `&mut Coin + amount` — safer for callers
2. **Return objects** instead of self-transferring — enables PTB composability
3. **Use `public(package)`** for internal functions — `public` signatures are permanent
4. **Test to 100% coverage** when feasible: `iota move test --coverage`
5. **Max object size: 250KB** — use dynamic fields for growing data
6. **Don't micro-optimize gas** — costs are bucketed, small changes don't matter

## Official Examples from the IOTA Repository

The [`examples/move/`](https://github.com/iotaledger/iota/tree/develop/examples/move) directory contains 24 ready-to-study packages:

| Example | What It Teaches |
|---------|----------------|
| `basics/counter` | Shared objects, owner-only access, basic tests |
| `coin` | Custom `Coin<T>` with OTW, `TreasuryCap`, minting |
| `flash_lender` | Hot Potato pattern, flash loans, `AdminCap` |
| `locked_stake` | System staking integration, `EpochTimeLock`, `VecMap` |
| `asset_tokenization` | Real-world asset tokenization |
| `abstract_iota_accounts` | Account abstraction patterns |
| `nft` | Basic NFT minting and transfer |
| `dynamic_fields` | Dynamic fields, heterogeneous collections |
| `hero` | Game character with equipment (RPG-style) |
| `token` | Closed-loop tokens with `Token<T>` |
| `transfer-to-object` | Receiving objects (Transfer-to-Object) |
| `trusted_swap` | Atomic swap between two parties |
| `random` | On-chain randomness |
| `vdf` | Verifiable delay functions |
| `reviews_rating` | Reviews and ratings system |
| `crypto` | Cryptographic primitives usage |
| `object_bound` | Soul-bound (non-transferable) objects |
| `errors` | Custom error codes and abort patterns |
| `entry_functions` | Entry function signatures |

### Counter (Shared Object Basics)

From `examples/move/basics/sources/counter.move`:

```move
module basics::counter {
    /// A shared counter.
    public struct Counter has key {
        id: UID,
        owner: address,
        value: u64
    }

    /// Create and share a Counter object.
    public fun create(ctx: &mut TxContext) {
        transfer::share_object(Counter {
            id: object::new(ctx),
            owner: tx_context::sender(ctx),
            value: 0
        })
    }

    /// Increment a counter by 1.
    public fun increment(counter: &mut Counter) {
        counter.value = counter.value + 1;
    }

    /// Set value (only runnable by the Counter owner)
    public fun set_value(counter: &mut Counter, value: u64, ctx: &TxContext) {
        assert!(counter.owner == ctx.sender(), 0);
        counter.value = value;
    }

    /// Delete counter (only runnable by the Counter owner)
    public fun delete(counter: Counter, ctx: &TxContext) {
        assert!(counter.owner == ctx.sender(), 0);
        let Counter {id, owner:_, value:_} = counter;
        id.delete();
    }
}
```

### Custom Coin (Fungible Token)

From `examples/move/coin/sources/my_coin.move`:

```move
module examples::my_coin {
    use iota::coin::{Self, TreasuryCap};

    /// OTW — name must match module name in UPPERCASE.
    public struct MY_COIN has drop {}

    fun init(witness: MY_COIN, ctx: &mut TxContext) {
        let (treasury, metadata) = coin::create_currency(
            witness, 6, b"MY_COIN", b"", b"",
            option::none(), ctx,
        );
        transfer::public_freeze_object(metadata);
        transfer::public_transfer(treasury, ctx.sender())
    }

    public fun mint(
        treasury_cap: &mut TreasuryCap<MY_COIN>,
        amount: u64,
        recipient: address,
        ctx: &mut TxContext,
    ) {
        let coin = coin::mint(treasury_cap, amount, ctx);
        transfer::public_transfer(coin, recipient)
    }
}
```

### Flash Lender (Hot Potato Pattern)

From `examples/move/flash_lender/sources/example.move`:

```move
module flash_lender::example {
    use iota::balance::{Self, Balance};
    use iota::coin::{Self, Coin};

    public struct FlashLender<phantom T> has key {
        id: UID,
        to_lend: Balance<T>,
        fee: u64,
    }

    /// Hot potato — no drop, no key, no store. Must be consumed by `repay`.
    public struct Receipt<phantom T> {
        flash_lender_id: ID,
        repay_amount: u64,
    }

    public struct AdminCap has key, store {
        id: UID,
        flash_lender_id: ID,
    }

    const ELoanTooLarge: u64 = 0;
    const EInvalidRepaymentAmount: u64 = 1;
    const ERepayToWrongLender: u64 = 2;

    public fun loan<T>(
        self: &mut FlashLender<T>, amount: u64, ctx: &mut TxContext
    ): (Coin<T>, Receipt<T>) {
        assert!(balance::value(&self.to_lend) >= amount, ELoanTooLarge);
        let loan = coin::take(&mut self.to_lend, amount, ctx);
        let receipt = Receipt {
            flash_lender_id: object::id(self),
            repay_amount: amount + self.fee,
        };
        (loan, receipt)
    }

    public fun repay<T>(
        self: &mut FlashLender<T>, payment: Coin<T>, receipt: Receipt<T>,
    ) {
        let Receipt { flash_lender_id, repay_amount } = receipt;
        assert!(object::id(self) == flash_lender_id, ERepayToWrongLender);
        assert!(coin::value(&payment) >= repay_amount, EInvalidRepaymentAmount);
        coin::put(&mut self.to_lend, payment);
    }
}
```

### Locked Staking

From `examples/move/locked_stake/sources/locked_stake.move`:

```move
module locked_stake::locked_stake {
    use iota::coin;
    use iota::balance::{Self, Balance};
    use iota::vec_map::{Self, VecMap};
    use iota::iota::IOTA;
    use iota_system::staking_pool::StakedIota;
    use iota_system::iota_system::{Self, IotaSystemState};
    use locked_stake::epoch_time_lock::{Self, EpochTimeLock};

    /// Locks IOTA tokens and stake objects until a given epoch.
    public struct LockedStake has key {
        id: UID,
        staked_iota: VecMap<ID, StakedIota>,
        iota: Balance<IOTA>,
        locked_until_epoch: EpochTimeLock,
    }

    public fun stake(
        ls: &mut LockedStake,
        iota_system: &mut IotaSystemState,
        amount: u64,
        validator_address: address,
        ctx: &mut TxContext,
    ) {
        let stake = iota_system::request_add_stake_non_entry(
            iota_system,
            coin::from_balance(balance::split(&mut ls.iota, amount), ctx),
            validator_address,
            ctx,
        );
        let id = object::id(&stake);
        vec_map::insert(&mut ls.staked_iota, id, stake);
    }
}
```

## IOTA Framework Modules

The [`iota-framework`](https://github.com/iotaledger/iota/tree/develop/crates/iota-framework/packages/iota-framework/sources) provides the standard library. Key modules:

| Module | Purpose |
|--------|---------|
| `coin.move` | `Coin<T>`, `TreasuryCap`, `CoinMetadata` |
| `balance.move` | `Balance<T>` — lightweight value-type for pools |
| `transfer.move` | Object transfer, sharing, freezing |
| `object.move` | `UID`, object creation, deletion |
| `dynamic_field.move` | Attach arbitrary fields to objects at runtime |
| `dynamic_object_field.move` | Like dynamic fields but for objects (preserves ID) |
| `bag.move` / `table.move` | Heterogeneous / homogeneous dynamic collections |
| `clock.move` | On-chain timestamps |
| `random.move` | On-chain verifiable randomness |
| `display.move` | Display standard for wallets/explorers |
| `package.move` | Package management, `Publisher` object |
| `token.move` | Closed-loop tokens with spend policies |
| `kiosk.move` | Commerce standard (marketplace primitives) |
| `bcs.move` | Binary Canonical Serialization |
| `event.move` | Emit events for off-chain indexing |
| `account_abstraction/` | Account abstraction framework |

## References

- [Create a Package](https://docs.iota.org/developer/getting-started/create-a-package)
- [IOTA Move CTF](https://docs.iota.org/developer/iota-move-ctf/introduction)
- [Dev Cheat Sheet](https://docs.iota.org/developer/dev-cheat-sheet)
- [Official Examples](https://github.com/iotaledger/iota/tree/develop/examples/move)
- [Framework Source](https://github.com/iotaledger/iota/tree/develop/crates/iota-framework/packages/iota-framework/sources)
