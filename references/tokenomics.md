# IOTA Rebased Tokenomics

## Table of Contents

- [Token: IOTA](#token-iota)
- [Economic Actors](#economic-actors)
- [Staking Rewards (Inflationary)](#staking-rewards-inflationary)
- [Transaction Fees (Deflationary)](#transaction-fees-deflationary)
- [Gas Pricing Mechanism](#gas-pricing-mechanism)
- [Storage Deposits](#storage-deposits)
- [Supply Dynamics](#supply-dynamics)
- [Sponsored Transactions](#sponsored-transactions)
- [Locked Token Unlocks](#locked-token-unlocks)
- [Governance](#governance)
- [Comparison with Other L1s](#comparison-with-other-l1s)
- [Future Plans](#future-plans)
- [References](#references)

## Token: IOTA

| Property | Value |
|----------|-------|
| **Name** | IOTA |
| **Decimals** | 9 (changed from 6 in Stardust — balances migrated ×1000) |
| **Smallest unit** | 1 Nano = 0.000000001 IOTA |
| **Initial supply** | ~4,600,000,000 IOTA (migrated from Stardust) |
| **Max supply** | None (dynamic — inflation vs burn) |
| **On-chain type** | `0x2::iota::IOTA` |

The IOTA token serves four purposes:

1. **Staking**: Participate in DPoS by delegating to validators
2. **Transaction Fees**: Pay gas for computation and storage
3. **Value Transfer**: Medium of exchange, unit of account, store of value
4. **Governance**: On-chain voting on protocol upgrades and parameters

All original Stardust IOTA tokens are accessible on the same addresses (hex format) — no manual migration needed for basic tokens.

## Economic Actors

Three main participants drive the IOTA economy:

- **Users**: Submit transactions, pay gas fees, interact with smart contracts
- **Validators**: Process and execute transactions, maintain the network
- **Delegators**: Stake IOTA tokens to validators, participate in governance

## Staking Rewards (Inflationary)

- **~767,000 IOTA minted per epoch** (~24 hours) as validator subsidy
- First-year inflation rate: **~6%**
- Rate **declines over time** because the mint amount is constant while total supply grows
- Rewards distributed proportionally to stake across validator pools

### Reward Distribution Per Epoch

At each epoch boundary:

1. **Total rewards** = validator subsidy (767k IOTA) + tips from user transactions
2. Rewards distributed to each validator pool proportional to its share of total stake
3. Each pool's rewards are adjusted by the **tallying rule** (μ_v): performant validators get μ=1, underperformers get μ<1
4. Validator takes **commission** (δ_v, max 20%) from pool rewards
5. Remaining rewards distributed to all stakers (including validator) proportionally

**Reward formulas:**

```
UserRewards_v = (1 - δ_v) × (1 - β_v) × μ_v × σ_v × TotalRewards
ValidatorRewards_v = (β_v + δ_v × (1 - β_v)) × μ_v × σ_v × TotalRewards

Where:
  δ_v = validator commission rate (max 20%)
  β_v = validator's own stake share of the pool
  μ_v = tallying rule score (1 = performant, <1 = penalized)
  σ_v = pool's share of total network stake
```

### Practical Example

If a validator pool holds 5% of total stake (σ=0.05), is performant (μ=1), and charges 10% commission (δ=0.1):
- Pool receives: 0.05 × 767,000 = **38,350 IOTA/epoch**
- Validator commission: 3,835 IOTA
- Distributed to delegators: 34,515 IOTA (proportional to their stake)

## Transaction Fees (Deflationary)

- Average transaction cost: **~0.005 IOTA**
- Computation fees are **burned** (removed from circulation)
- Higher network usage = more fees burned = more deflationary pressure

### Fee Calculation

```
computation_fee = computation_units × gas_price
                = computation_units × (reference_gas_price + tip)

storage_fee = storage_units × storage_price

net_gas_fees = computation_fee + storage_deposit - storage_rebate
```

### Gas Price

- **Reference gas price**: Currently fixed at the `base_gas_price` protocol parameter
- **Initial value**: 1,000 Nanos per computation unit
- Users must specify `gas_price >= reference_gas_price`; any excess is treated as a **tip**
- Tips go to validators/delegators as additional rewards

## Gas Pricing Mechanism

### Computation Units (Bucketed)

Gas computation uses a **bucketing/step approach**:

- Similar transactions fall into the same bucket → same cost
- Smallest bucket: **1,000 computation units**
- Step between buckets: **1,000 units**
- Largest bucket: **5,000,000 units** (transactions exceeding this abort)

**Key insight**: Don't micro-optimize gas — only drastic changes that move you to a different bucket matter. If you're in the lowest bucket already, you can't get cheaper.

### Storage Units (Linear)

- **100 storage units per byte**
- Example: 25 bytes = 2,500 storage units; 75 bytes = 7,500 units
- Storage deposits are **100% rebatable** when objects are deleted

### Gas Budget

Users submit transactions with a gas budget cap:
```
gas_budget >= max(computation_fees, net_gas_fees)
```
If insufficient: transaction fails and a portion of the budget is charged.

## Storage Deposits

- Storing data on-chain requires a **redeemable deposit**
- Similar concept to Stardust's storage deposit
- **100% returned** when the stored object is deleted
- Incentivizes users to clean up unused objects

## Supply Dynamics

```
New Supply = Previous Supply + Staking Rewards (767k/epoch) - Burned Fees
```

```
Year 1:  ~4.6B + ~280M rewards - burns = net ~6% inflation (if low usage)
Year 2:  ~4.88B + ~280M rewards - burns = net ~5.7% inflation
Year 5:  ~5.7B + ~280M rewards - burns = net ~4.9% inflation
Year 10: ~7.1B + ~280M rewards - burns = net ~3.9% inflation
```

The inflation rate asymptotically approaches 0% as supply grows. With high network usage, **burns can exceed minting**, making IOTA deflationary.

```
Conceptual Supply Over Time:

Supply ▲
       │          ╱ Low usage (inflationary)
       │        ╱
       │      ╱───── Medium usage (near-neutral)
       │    ╱
       │  ╱  ╲─── High usage (deflationary)
       │╱
       └──────────────────────► Time
       4.6B
```

## Sponsored Transactions

Apps can pay transaction fees on behalf of users:

- Users don't need to hold IOTA to interact with dApps
- Great for onboarding new users from Web2
- App developer covers gas costs
- Implemented at the protocol level (not a workaround)

## Locked Token Unlocks

Tokens locked during the Stardust 2023 upgrade continue unlocking on their original schedule. Importantly:

- Locked tokens **can** be delegated to validators
- You earn staking rewards even while tokens are locked
- Unlocking happens automatically at the scheduled time

## Governance

- IOTA tokens grant the right to participate in **on-chain governance**
- Token holders vote on protocol upgrades and parameter changes
- All tokenomics parameters (mint rate, commission cap, validator seats) are **governance-adjustable**
- Protocol upgrades require 2f+1 validator agreement

## Comparison with Other L1s

| Property | IOTA Rebased | Ethereum | Sui | Solana |
|----------|-------------|----------|-----|--------|
| **Consensus** | DPoS + Mysticeti BFT | PoS + Gasper | DPoS + Mysticeti | PoH + Tower BFT |
| **VM** | Move VM | EVM | Move VM | SVM (Sealevel) |
| **Inflation** | ~6% Y1, declining | ~0.5-1% | ~varies | ~5.5%, declining |
| **Fee mechanism** | Burned | Base burned + tips | Burned | 50% burned + 50% validators |
| **Staking yield** | ~6% initial | ~3-4% | ~varies | ~6-7% |
| **Min validator stake** | 2M IOTA | 32 ETH | ~30M SUI | ~vote-weighted |
| **Max validators** | 150 | ~900k | ~100+ | ~1500+ |
| **Sponsored txs** | ✅ Native | ❌ (ERC-4337) | ✅ Native | ❌ |
| **Storage model** | Rebatable deposit | Permanent (no rebate) | Rebatable deposit | Rent-based |

## Future Plans

- **Dynamic gas pricing**: Protocol-level algorithms will replace fixed pricing, factoring in congestion and shared object popularity
- **Starfish consensus**: Improved BFT with better Byzantine resilience (already on devnet)
- **Decentralized treasury**: Protocol may direct some fees to ecosystem funding
- **All parameters are governance-adjustable** through on-chain voting

## References

- [IOTA Tokenomics Overview](https://docs.iota.org/about-iota/tokenomics/)
- [IOTA Token](https://docs.iota.org/about-iota/tokenomics/iota-token)
- [Gas in IOTA](https://docs.iota.org/about-iota/tokenomics/gas-in-iota)
- [Gas Pricing](https://docs.iota.org/about-iota/tokenomics/gas-pricing)
- [Proof of Stake](https://docs.iota.org/about-iota/tokenomics/proof-of-stake)
- [Validators & Staking](https://docs.iota.org/about-iota/tokenomics/validators-staking)
- [IOTA Rebased Technical View](https://blog.iota.org/iota-rebased-technical-view/)
