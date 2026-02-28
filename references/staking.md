# Staking on IOTA Rebased

## Table of Contents

- [How DPoS Works](#how-dpos-works)
- [Staking Rewards](#staking-rewards)
- [Epochs](#epochs)
- [Staking Pool Mechanics](#staking-pool-mechanics)
- [Delegating via Wallet (Step by Step)](#delegating-via-wallet-step-by-step)
- [Delegating via CLI (Step by Step)](#delegating-via-cli-step-by-step)
- [Choosing a Good Validator](#choosing-a-good-validator)
- [Rewards Calculator](#rewards-calculator)
- [Locked Token Staking](#locked-token-staking)
- [Risks and Penalties](#risks-and-penalties)
- [Becoming a Validator](#becoming-a-validator)
- [Validator Committee and Quorums](#validator-committee-and-quorums)
- [References](#references)

## How DPoS Works

IOTA Rebased uses Delegated Proof of Stake (DPoS):

1. **Validators** run nodes and process transactions
2. **Delegators** stake IOTA tokens to validators they trust
3. Both earn **staking rewards** proportional to stake
4. The validator set is **fixed during each epoch** (~24 hours)
5. At epoch boundaries: rewards distributed, validator set may change, staking/unstaking processed

## Staking Rewards

- **~767,000 IOTA minted per epoch** as validator subsidy
- Plus any **tips** from user transactions
- First-year inflation: ~6%, declining over time (fixed mint rate, growing supply)
- Rewards are distributed proportionally to stake, minus validator commission
- **Net positive for stakers**: rewards typically exceed transaction fees spent
- Staking rewards **compound automatically** — earned IOTA immediately counts as stake

## Epochs

- Duration: **~24 hours**
- Validator set fixed during each epoch
- At epoch boundary:
  - Staking rewards distributed
  - Pending stake/unstake requests processed
  - Validator set updated (new validators join, underperformers removed)
  - Protocol upgrades can activate (if 2f+1 validators agree)

## Staking Pool Mechanics

Each validator maintains a **staking pool** tracked by exchange rates computed at epoch boundaries.

### How Exchange Rates Work

1. When you deposit IOTA in epoch E, your tokens are converted to **staked tokens** at epoch E's exchange rate
2. As the pool earns rewards, the exchange rate **appreciates**
3. At epoch E', your staked tokens are worth more IOTA
4. All tokens in the pool (original stake + rewards) immediately count as stake → **automatic compounding**

The staking pool is implemented as a system-level smart contract: [`staking_pool.move`](https://github.com/iotaledger/iota/blob/develop/crates/iota-framework/packages/iota-system/sources/staking_pool.move)

### Reward Distribution Formula

```
rewards(i, E) = [stakedTokens(i, E) / Σ stakedTokens(all, E)] × totalRewards(E)
```

Then adjusted for:
- Validator performance (tallying rule μ_v)
- Validator commission (δ_v, max 20%)

## Delegating via Wallet (Step by Step)

### Prerequisites
- IOTA Wallet browser extension ([Chrome Web Store](https://chrome.google.com/webstore/detail/iota-wallet/lfhohhmmcplfddhojblhpbblpbcoecha))
- IOTA tokens in your wallet

### Steps

1. **Open IOTA Wallet** → Click on "Staking" tab
2. **Browse validators** → View their commission rates, total stake, and performance
3. **Select a validator** → Click "Stake"
4. **Enter amount** → Specify how many IOTA to delegate
5. **Confirm transaction** → Sign the staking transaction
6. **Done!** → Rewards accrue starting next epoch

### Unstaking

1. Go to "Staking" tab → View your active stakes
2. Click "Unstake" on the delegation you want to withdraw
3. Confirm the transaction
4. **Tokens return at the next epoch boundary** (up to ~24h wait)

## Delegating via CLI (Step by Step)

### List Available Validators

```bash
iota client validators
```

### Stake IOTA to a Validator

```bash
# First, find your coin object IDs
iota client gas

# Stake to a validator
iota client call \
    --package 0x3 \
    --module iota_system \
    --function request_add_stake \
    --args 0x5 <COIN_OBJECT_ID> <VALIDATOR_ADDRESS> \
    --gas-budget 10000000
```

**Parameters:**
- `0x5` — the IOTA System State object (fixed address)
- `<COIN_OBJECT_ID>` — the Coin object you want to stake
- `<VALIDATOR_ADDRESS>` — the validator's address (from `iota client validators`)

### Check Your Stakes

```bash
iota client objects --type StakedIota
```

### Unstake

```bash
iota client call \
    --package 0x3 \
    --module iota_system \
    --function request_withdraw_stake \
    --args 0x5 <STAKED_IOTA_OBJECT_ID> \
    --gas-budget 10000000
```

## Choosing a Good Validator

When selecting a validator to delegate to, consider these metrics:

| Metric | What to Look For | Why It Matters |
|--------|-----------------|----------------|
| **Commission rate** | Lower is better for delegators (max 20%) | Directly reduces your share of rewards |
| **Total stake** | Well-established, but not too dominant | Very large validators centralize the network |
| **Performance (μ_v)** | Should be 1.0 (full performance) | Underperformers get slashed rewards |
| **Uptime** | As close to 100% as possible | Offline validators miss blocks and hurt rewards |
| **Community reputation** | Known team, transparent operations | Reduces risk of malicious behavior |
| **Own stake (β_v)** | Higher is better (skin in the game) | Validator is more incentivized to perform well |

### Red Flags
- ❌ Very high commission (>15%)
- ❌ No self-stake (no skin in the game)
- ❌ Unknown/anonymous operator with no track record
- ❌ Frequently changing commission rates upward

## Rewards Calculator

### Approximate Annual Yield Formula

```
annual_yield ≈ (767,000 × 365 × your_share × (1 - commission)) / your_stake

Where:
  your_share = your_stake / total_network_stake
  commission = validator's commission rate (e.g., 0.10 for 10%)
```

### Example Calculation

Assumptions:
- You stake: **10,000 IOTA**
- Total network stake: **2,000,000,000 IOTA**
- Validator commission: **10%**
- Validator performance: **μ = 1.0**

```
Daily rewards per epoch = 767,000 × (10,000 / 2,000,000,000) × (1 - 0.10)
                        = 767,000 × 0.000005 × 0.90
                        = 3.45 IOTA per day

Annual yield = 3.45 × 365 = ~1,259 IOTA/year
Annual rate = 1,259 / 10,000 = ~12.6%
```

> ⚠️ Actual yields depend on total network stake, validator performance, tips, and compounding effects. Early epochs with lower total stake yield higher returns.

## Locked Token Staking

Tokens from the Stardust migration with time-locks:

- **Can be delegated** to validators while still locked
- Earn **full staking rewards** as if they were unlocked
- Tokens remain locked until their original unlock schedule
- Rewards earned are **unlocked** and can be used freely
- This was specifically designed to not penalize long-term holders

## Risks and Penalties

### Is There Slashing?

IOTA uses a **soft penalty** model rather than hard slashing:

- **No token slashing** — your staked tokens are never destroyed as punishment
- **Performance penalties** — underperforming validators receive reduced rewards (μ_v < 1)
- **Reputation loss** — poor performance leads to delegators moving their stake away
- **Removal from validator set** — if stake falls below thresholds (see below)

### Validator Removal Thresholds

| Condition | Action |
|-----------|--------|
| Stake drops below **1.5M IOTA** | 7-epoch grace period to regain stake |
| Stake drops below **1M IOTA** | Removed at end of current epoch |
| Persistent underperformance | Delegators leave → stake naturally decreases |

### Risk Summary for Delegators

| Risk | Severity | Mitigation |
|------|----------|------------|
| Validator underperformance | Low | Reduced rewards (not token loss); switch validators |
| Validator goes offline | Low | Rewards reduced; stake can be moved next epoch |
| Token lock during epoch | Minimal | Staked tokens locked for ~24h max per epoch |
| Protocol bugs | Very low | Move VM's static verification + formal verification |

## Becoming a Validator

### Requirements

| Resource | Minimum |
|----------|---------|
| **IOTA Stake** | 2,000,000 IOTA (can include delegations) |
| **RAM** | 128 GB |
| **CPU** | 24 cores / 48 vCPUs |
| **Storage** | 4 TB NVMe SSD |
| **Network** | 1 Gbps uplink |

### Key Points

- Max **150 validator seats** (initially)
- Validators set a **commission rate** (% of delegator rewards kept, max 20%)
- IOTA's improved selection: new candidates with more stake **can replace** existing validators with less stake
- Validators must maintain uptime and performance
- A candidate must accumulate 2M IOTA stake before requesting to join

### Setup Guide

For detailed validator setup instructions, see: [IOTA Validator Node Configuration](https://docs.iota.org/operator/validator-node/configuration)

## Validator Committee and Quorums

- Validators form the **Consensus Committee** that executes Mysticeti/Starfish
- A **quorum** = validators with combined voting power > 2/3 of total
- Transactions are committed only when signed by a quorum (**certificate**)
- The validator set is **fixed during each epoch** for stability
- Validator voting power is proportional to stake
- Each validator independently verifies transactions and signs them

### Write Request Flow

1. Client sends transaction to quorum of validators
2. Validators verify and sign → client collects signatures into **certificate**
3. Client submits certificate to validators → they execute the transaction
4. Execution either succeeds (all effects applied) or aborts (only gas debited)

## References

- [Validators & Staking](https://docs.iota.org/about-iota/tokenomics/validators-staking)
- [Proof of Stake](https://docs.iota.org/about-iota/tokenomics/proof-of-stake)
- [Validator Committee](https://docs.iota.org/about-iota/iota-architecture/validator-committee)
- [Epochs and Reconfiguration](https://docs.iota.org/about-iota/iota-architecture/epochs)
- [Validator Node Overview](https://docs.iota.org/operator/validator-node/overview)
- [Validator Node Configuration](https://docs.iota.org/operator/validator-node/configuration)
- [Gas Pricing / Tallying Rule](https://docs.iota.org/about-iota/tokenomics/gas-pricing)
