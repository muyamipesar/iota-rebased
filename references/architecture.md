# IOTA Rebased Architecture

## Table of Contents

- [Overview](#overview)
- [Move VM](#move-vm)
- [Object Model](#object-model)
- [Programmable Transaction Blocks (PTBs)](#programmable-transaction-blocks-ptbs)
- [Transaction Lifecycle](#transaction-lifecycle)
- [Mysticeti Consensus](#mysticeti-consensus)
- [Starfish Consensus (Next Generation)](#starfish-consensus-next-generation)
- [Delegated Proof of Stake (DPoS)](#delegated-proof-of-stake-dpos)
- [Epochs and Reconfiguration](#epochs-and-reconfiguration)
- [Checkpoints](#checkpoints)
- [Cryptography](#cryptography)
- [Security Model](#security-model)
- [Protocol Upgrades](#protocol-upgrades)
- [Key Differences from Sui](#key-differences-from-sui)
- [References](#references)

## Overview

IOTA Rebased is built on a protocol derived from Sui, with custom improvements. The core components are:

1. **Move Virtual Machine (Move VM)** — execution engine for L1 smart contracts
2. **Object Model** — data/state representation (everything is an object)
3. **Mysticeti/Starfish Consensus** — DAG-based BFT consensus for shared objects
4. **Delegated Proof of Stake (DPoS)** — validator selection and network security
5. **Programmable Transaction Blocks** — batched atomic operations

## Move VM

The Move VM is the execution engine for all L1 smart contracts. Key properties:

- **Resource-oriented**: Digital assets are first-class resources that can't be accidentally duplicated or destroyed
- **Static verification**: Bytecode verified at publish time — many bugs caught before execution
- **Formal verification support**: Mathematical proofs of contract correctness
- **Move 2024 edition**: Method syntax (`v.push_back(x)`), enums, macros, index syntax (see [move-basics.md](move-basics.md))

Move code is organized into **packages** containing **modules**. Modules define **structs** (object types) and **functions**.

## Object Model

Everything on IOTA Rebased is an **object** with a unique 32-byte ID. Three ownership types:

### Owned Objects
- Belong to a specific address
- Only the owner can use them in transactions
- **Can be processed in parallel** (no consensus needed for single-owner txs)
- This is the "fast path" — most transactions use owned objects

### Shared Objects
- Accessible by anyone
- Require consensus ordering (Mysticeti/Starfish)
- Used for DEXes, shared state, AMM pools, etc.
- Higher latency than owned objects due to consensus requirement

### Immutable Objects
- Cannot be modified or deleted
- Published packages are immutable objects
- No ownership — anyone can read

### Object Abilities

Move structs can have abilities: `key` (can be stored as object), `store` (can be nested), `copy`, `drop`.

An object must have `key` ability and a `UID` field:

```move
public struct MyObject has key, store {
    id: UID,
    value: u64,
}
```

### Max Object Size

Objects have a **maximum size of 250KB**. For larger or growing data, use dynamic fields (Tables, Bags).

### Dynamic Fields

Objects can have dynamic fields added at runtime — useful for extensible data and large collections:

```move
// Add a dynamic field
dynamic_field::add(&mut obj.id, key, value);

// Read a dynamic field
let val = dynamic_field::borrow(&obj.id, key);
```

## Programmable Transaction Blocks (PTBs)

PTBs allow batching multiple operations in a single atomic transaction. This is one of IOTA's most powerful features.

### How PTBs Work

A PTB contains a list of **commands** executed sequentially within one transaction:

```
Command 1: Split coin into [1 IOTA, 2 IOTA]
Command 2: Transfer 1 IOTA to Alice
Command 3: Call DEX swap with 2 IOTA
Command 4: Transfer swap result to Bob
```

### Properties

- **Atomic**: All commands succeed or all fail — no partial execution
- **Composable**: Output of one command feeds into the next
- **Efficient**: Single transaction, single gas payment
- **No intermediary contracts needed**: Complex DeFi flows without helper contracts

### PTB Example (TypeScript SDK)

```typescript
import { Transaction } from '@iota/iota-sdk/transactions';

const tx = new Transaction();

// Split coin
const [coin1, coin2] = tx.splitCoins(tx.gas, [1_000_000_000, 2_000_000_000]);

// Transfer coin1 to Alice
tx.transferObjects([coin1], 'ALICE_ADDRESS');

// Call a smart contract with coin2
tx.moveCall({
    target: '0xPACKAGE::module::function',
    arguments: [coin2],
});
```

### PTB Example (CLI)

```bash
iota client ptb \
    --split-coins gas "[1000000000, 2000000000]" \
    --transfer-objects "[result_0_0]" ALICE_ADDRESS \
    --move-call 0xPACKAGE::module::function result_0_1 \
    --gas-budget 10000000
```

## Transaction Lifecycle

Detailed flow of a transaction from creation to finality:

```
┌─────────────┐     ┌──────────────┐     ┌───────────────┐
│ 1. User      │────▶│ 2. Full Node  │────▶│ 3. Validators  │
│ Creates &    │     │ Validates &   │     │ Verify & sign  │
│ signs tx     │     │ distributes   │     │ the tx         │
└─────────────┘     └──────────────┘     └───────┬───────┘
                                                  │
                                          ┌───────▼───────┐
                                          │ 4. Client      │
                                          │ assembles      │
                                          │ CERTIFICATE    │
                                          │ (>2/3 sigs)    │
                                          └───────┬───────┘
                                                  │
                    ┌─────────────────────────────┤
                    │                             │
            ┌───────▼───────┐            ┌────────▼────────┐
            │ OWNED OBJECTS  │            │ SHARED OBJECTS   │
            │ Fast path:     │            │ Consensus path:  │
            │ Execute now!   │            │ Mysticeti orders │
            │ No consensus   │            │ then execute     │
            └───────┬───────┘            └────────┬────────┘
                    │                             │
                    └─────────────┬───────────────┘
                                  │
                          ┌───────▼───────┐
                          │ 5. FINALITY    │
                          │ Effects cert   │
                          │ + Checkpoint   │
                          └───────────────┘
```

### Step-by-Step

1. **User creates transaction** — wallet/app builds and signs with private key
2. **Full node validates** — checks structure, balance, distributes to validators
3. **Validators verify and sign** — each validator independently checks and returns signed vote
4. **Client assembles certificate** — collects >2/3 validator signatures into a certificate
5. **Certificate submitted to validators**:
   - **Owned objects only** → execute immediately (fast path, sub-second)
   - **Shared objects** → route through Mysticeti/Starfish consensus for ordering, then execute
6. **Execution** — either succeeds (all effects) or aborts (only gas deducted)
7. **Effect certificate** — client can collect execution confirmations from >2/3 validators
8. **Checkpoint** — transaction included in next periodic checkpoint for permanent record

### Finality

- **Owned-object transactions**: Sub-second finality (fast path)
- **Shared-object transactions**: ~2-3 seconds (consensus ordering)
- **Checkpoint inclusion**: Asynchronous, for permanent ledger record

## Mysticeti Consensus

Mysticeti is a DAG-based BFT consensus protocol for ordering transactions involving shared objects.

### Key Features

- **Parallel block proposals**: Multiple validators propose blocks simultaneously, maximizing bandwidth
- **Three-round finality**: Blocks reach finality in only 3 rounds of communication (theoretical minimum for BFT)
- **Optimized voting**: Validators vote and certify blocks in parallel, reducing median and tail latencies
- **Fault tolerance**: Resilient even if some validators are offline/crashed, without substantially impacting commit latency
- **Uncertified DAG structure**: Efficient, low-overhead block propagation

### Observed Limitations

Testing revealed scenarios where Mysticeti performance may degrade:
- Disrupted/blocked connections between validators
- Network latency patterns violating triangle inequality assumptions
- Validators behaving adversarially or delaying messages

These limitations motivated the development of Starfish.

🔗 Reference: [Mysticeti Paper](https://arxiv.org/pdf/2310.14821)

## Starfish Consensus (Next Generation)

Starfish is the next-generation BFT consensus protocol, designed to address Mysticeti's limitations. **Already deployed on Devnet**, pending validator acceptance for mainnet upgrade.

### Key Improvements Over Mysticeti

| Feature | Mysticeti | Starfish |
|---------|-----------|----------|
| **Block structure** | Monolithic | Decoupled headers + payloads |
| **Header dissemination** | Standard broadcast | Cordial dissemination (single connection sufficient) |
| **Transaction data** | Full broadcast | Reed-Solomon erasure coding (linear cost) |
| **Byzantine resilience** | Good | Superior (deterministic performance) |
| **Network assumptions** | Triangle inequality | No special assumptions |

### How Starfish Works

1. **Decoupled headers and payloads**: Block metadata separated from transaction data
2. **Cordial dissemination**: Even with multiple failed routes, a single functioning connection propagates headers — no retries needed
3. **Erasure coding**: Transactions encoded as Reed-Solomon shards — nodes reconstruct from partial data, eliminating redundant broadcasts

🔗 Reference: [Starfish Paper](https://eprint.iacr.org/2025/567)

## Delegated Proof of Stake (DPoS)

- Up to **150 validator seats**
- **Minimum stake: 2M IOTA** (can include delegated tokens)
- Validators earn staking rewards (~767k IOTA minted per epoch)
- **Delegators** stake tokens to validators and share rewards (minus commission, max 20%)
- IOTA improves on Sui's validator selection: new candidates with higher stake can replace existing validators

### Hardware Requirements (Mainnet Validators)

| Resource | Requirement |
|----------|-------------|
| RAM | 128 GB |
| CPU | 24 cores / 48 vCPUs |
| Storage | 4 TB NVMe SSD |
| Network | 1 Gbps uplink |

See [staking.md](staking.md) for complete details on validators and staking.

## Epochs and Reconfiguration

The network operates in **epochs** of approximately **24 hours**.

### During an Epoch
- Validator set is **fixed**
- Transactions processed normally by the consensus committee
- Staked tokens are **locked** for the duration

### At Epoch Boundary (Reconfiguration)

1. **Finalize transactions**: Generate final checkpoint with complete transaction history
2. **Distribute rewards**: 767k IOTA subsidy + tips distributed to staking pools
3. **Process staking changes**: Pending stake/unstake requests take effect
4. **Update validator set**: Add new validators, remove underperformers
5. **Protocol upgrades**: If 2f+1 validators agree, new protocol version activates

### Epoch Numbering

Each transaction includes an epoch identifier. Transactions can optionally specify an **expiration epoch** after which they become invalid.

> ⚠️ Avoid submitting time-sensitive transactions immediately before epoch transitions.

## Checkpoints

Checkpoints define the history of the IOTA blockchain (analogous to blocks in traditional chains):

- Created periodically during consensus commits
- Contain a set of finalized transactions
- Include validator agreement data and state updates
- Used to drive validator reconfiguration at epoch transitions
- Full nodes and validators verify checkpoints before trusting them
- Provide the stable point of reference for the ledger state

## Cryptography

IOTA supports multiple cryptographic schemes with **cryptographic agility** as a core design principle:

### Supported Signature Schemes

| Scheme | Use Case |
|--------|----------|
| **Ed25519** | Default, fastest, same as Stardust |
| **ECDSA Secp256k1** | Bitcoin-compatible |
| **ECDSA Secp256r1** | WebAuthn/passkey compatible |
| **Multisig** | Multi-key authorization (combining any of the above) |

### Key Features

- **BIP-32/44/39 compliant**: Standard HD wallet key derivation
- **Intent signing**: Signatures commit to an intent message (domain separator), not raw bytes
- **Offline signing**: Supported for air-gapped devices
- **Checkpoint verification**: Cryptographic proof that checkpoints were created by the validator committee

### On-Chain Cryptography (in Move)

Smart contracts have access to cryptographic primitives:
- Hash functions (SHA-256, SHA3-256, Blake2b, Keccak-256)
- Signature verification (Ed25519, ECDSA)
- BLS12-381 operations (for advanced crypto like ZK proofs)

See [Cryptography docs](https://docs.iota.org/developer/cryptography) for details.

## Security Model

IOTA's security architecture is multi-layered:

- **Byzantine Fault Tolerance**: Network secure as long as >2/3 of stake is honest
- **Certificate mechanism**: Transactions committed only with >2/3 validator signatures
- **Static verification**: Move bytecode verified at publish time
- **Resource types**: Assets can't be duplicated, lost, or double-spent at the language level
- **Object ownership**: Clear access control — only owners can touch owned objects
- **No reentrancy**: Move's design eliminates reentrancy attacks
- **Equivocation protection**: Signing concurrent txs on the same owned object locks it until epoch end

## Protocol Upgrades

The IOTA protocol is frequently extended with new functionality:

- Upgrades require **2f+1 validator agreement** (BFT supermajority)
- Processed at epoch boundaries
- Can include: security patches, new features, Move framework updates
- Parameters like gas prices, validator limits, and rewards are governance-adjustable

## Key Differences from Sui

While based on Sui, IOTA Rebased introduces:

| Feature | Sui | IOTA Rebased |
|---------|-----|-------------|
| **Consensus** | Mysticeti | Mysticeti → Starfish (upgrade path) |
| **Byzantine resilience** | Standard | Enhanced (Starfish) |
| **Gas pricing** | Validator-influenced | Protocol-level (fairer) |
| **Validator selection** | Fixed set | Dynamic (higher-stake candidates can replace) |
| **Staking rewards** | Temporary subsidies | Permanent (767k/epoch) |
| **L2 EVM** | No native L2 | ISC (IOTA Smart Contracts) for EVM |
| **Legacy migration** | N/A | Stardust UTXO → Move objects |
| **Native token** | SUI | IOTA |
| **Framework prefix** | `sui::` | `iota::` |
| **Max validators** | ~100+ | 150 |

## References

- [IOTA Architecture Overview](https://docs.iota.org/about-iota/iota-architecture/)
- [Transaction Lifecycle](https://docs.iota.org/about-iota/iota-architecture/transaction-lifecycle)
- [Consensus](https://docs.iota.org/about-iota/iota-architecture/consensus)
- [Epochs and Reconfiguration](https://docs.iota.org/about-iota/iota-architecture/epochs)
- [Validator Committee](https://docs.iota.org/about-iota/iota-architecture/validator-committee)
- [IOTA Security](https://docs.iota.org/about-iota/iota-architecture/iota-security)
- [Protocol Upgrades](https://docs.iota.org/about-iota/iota-architecture/protocol-upgrades)
- [Cryptography](https://docs.iota.org/developer/cryptography)
- [Mysticeti Paper](https://arxiv.org/pdf/2310.14821)
- [Starfish Paper](https://eprint.iacr.org/2025/567)
- [IOTA Rebased Technical View](https://blog.iota.org/iota-rebased-technical-view/)
- [Main repo](https://github.com/iotaledger/iota)

## Main Repository Structure

The core node implementation lives at [github.com/iotaledger/iota](https://github.com/iotaledger/iota) — a large Rust monorepo with 60+ crates.

### Key Crates

| Crate | Purpose |
|-------|---------|
| `iota-node` | Full node binary |
| `iota-core` | Core protocol logic (execution, state, authority) |
| `iota-framework` | Move framework packages (stdlib, IOTA-specific modules) |
| `iota-types` | Core types shared across all crates |
| `iota-json-rpc` | JSON-RPC API implementation |
| `iota-graphql-rpc` | GraphQL API implementation |
| `iota-indexer` | On-chain data indexing |
| `iota-benchmark` | Performance benchmarking tools |
| `iota-cluster-test` | Integration/cluster testing |
| `iota-adapter-transactional-tests` | Move transactional test framework |
| `iota-analytics-indexer` | Analytics data pipeline |
| `iota-archival` | Archive node support |

### Building from Source

```bash
# Install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
rustup toolchain install nightly --component rustfmt --allow-downgrade

# Clone and build
git clone https://github.com/iotaledger/iota.git
cd iota
cargo build --release

# Install IOTA CLI
cargo install --locked --bin iota --path crates/iota
```

### Move Framework Location

The IOTA Move framework is included in the main repo:
- `iota-framework/packages/iota-framework/` — Core modules (`iota::coin`, `iota::transfer`, `iota::object`, etc.)
- `iota-framework/packages/iota-system/` — System modules (staking, validators, epoch management)
- `iota-framework/packages/move-stdlib/` — Standard Move library
