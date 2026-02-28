# SDKs and Tools for IOTA Rebased

## IOTA CLI

The primary development tool. Install via Cargo:

```bash
cargo install --locked --git https://github.com/iotaledger/iota.git --branch mainnet iota
```

### Essential Commands

```bash
# Environment
iota client                          # Connect to network
iota client envs                     # List environments
iota client new-env --alias testnet --rpc https://api.testnet.iota.cafe
iota client switch --env testnet
iota client active-address           # Current address

# Keys
iota keytool list                    # List all addresses
iota client new-address --key-scheme ed25519  # Generate new address

# Move development
iota move new <package_name>         # Create package
iota move build                      # Build
iota move test                       # Run tests
iota move test --coverage            # With coverage
iota move coverage source --module <mod>  # Show uncovered lines

# Publishing & calling
iota client publish --gas-budget 100000000
iota client call --package 0x... --module ... --function ... --args ... --gas-budget 10000000

# Serialize for offline signing
iota client call ... --serialize-output  # Outputs base64 for signing
```

## TypeScript SDK

```bash
npm install @iota/iota-sdk
```

### Basic Usage

```typescript
import { IotaClient, getFullnodeUrl } from '@iota/iota-sdk/client';
import { Transaction } from '@iota/iota-sdk/transactions';
import { Ed25519Keypair } from '@iota/iota-sdk/keypairs/ed25519';

// Connect
const client = new IotaClient({ url: getFullnodeUrl('testnet') });

// Query objects
const objects = await client.getOwnedObjects({ owner: '0xYOUR_ADDRESS' });

// Build and execute transaction
const tx = new Transaction();
tx.moveCall({
    target: '0xPACKAGE::module::function',
    arguments: [tx.pure.u64(100)],
});

const keypair = Ed25519Keypair.fromSecretKey(secretKey);
const result = await client.signAndExecuteTransaction({
    signer: keypair,
    transaction: tx,
});
```

### dApp Kit (React)

```bash
pnpm create @iota/dapp --template react-client-dapp
```

Provides wallet connection, transaction signing, and React hooks out of the box.

- [dApp Kit docs](https://docs.iota.org/developer/ts-sdk/dapp-kit/)

## Rust SDK

```toml
# Cargo.toml
[dependencies]
iota-sdk = { git = "https://github.com/iotaledger/iota.git", branch = "mainnet" }
```

- [Rust SDK docs](https://docs.iota.org/developer/references/rust-sdk)

## GraphQL API

IOTA provides a GraphQL API for querying on-chain data:

```
Mainnet:  https://graphql.mainnet.iota.cafe
Testnet:  https://graphql.testnet.iota.cafe
Devnet:   https://graphql.devnet.iota.cafe
Local:    http://127.0.0.1:8000
```

### Example Query

```graphql
query {
    object(address: "0xOBJECT_ID") {
        objectId
        version
        digest
        owner {
            ... on AddressOwner {
                owner { address }
            }
        }
    }
}
```

- [GraphQL docs](https://docs.iota.org/developer/getting-started/graphql-rpc)

## JSON-RPC API

Standard JSON-RPC endpoint for programmatic access:

```bash
curl -X POST https://api.testnet.iota.cafe \
  -H 'Content-Type: application/json' \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "iota_getOwnedObjects",
    "params": ["0xYOUR_ADDRESS", null, null, null]
  }'
```

## Local Development Network

```bash
RUST_LOG="off,iota_node=info" cargo run --bin iota start --force-regenesis --with-faucet
```

This starts a local network with:
- RPC: `http://127.0.0.1:9000`
- Faucet: `http://127.0.0.1:9123/gas`
- GraphQL: `http://127.0.0.1:8000`

## IDE Support

- **VS Code**: Install the Move Analyzer extension for syntax highlighting, error checking, and go-to-definition
- **IntelliJ**: Move language plugin available
- **move-analyzer LSP**: Ships with the IOTA CLI tools

## Recommended Development Setup

1. Install Rust + Cargo
2. Install IOTA CLI
3. Install VS Code + Move Analyzer extension
4. Install Node.js + pnpm (for TypeScript SDK / dApp Kit)
5. Set up testnet environment in CLI

## Rust SDK (`iota-sdk` crate)

The official Rust SDK lives at [github.com/iotaledger/iota-rust-sdk](https://github.com/iotaledger/iota-rust-sdk). It's modular — use only the features you need:

```toml
[dependencies]
iota-sdk = { version = "0.x", features = ["types", "crypto", "graphql", "txn-builder"] }
```

### Crate Structure

| Crate | Purpose |
|-------|---------|
| `iota-sdk` | Meta-crate re-exporting all sub-crates |
| `iota-sdk-types` | Core types (addresses, objects, digests) |
| `iota-sdk-crypto` | Cryptographic primitives (key pairs, signing) |
| `iota-sdk-graphql-client` | GraphQL client for querying on-chain data |
| `iota-sdk-transaction-builder` | Build and serialize transactions |

### Basic Usage (Rust)

```rust
use iota_sdk::types;       // Core types
use iota_sdk::crypto;      // Key management
use iota_sdk::graphql_client; // Query chain
use iota_sdk::transaction_builder; // Build txs
```

The SDK targets wasm compatibility where possible, keeping a minimal dependency footprint.

**Docs:** [docs.rs/iota-sdk](https://docs.rs/iota-sdk)

## Go SDK (`iota-sdk-go`)

Go bindings for the IOTA SDK via UniFFI (Rust → Go FFI). Pre-built native libraries for macOS, Linux, Windows (x86_64 + ARM64).

```bash
go get github.com/iotaledger/iota-sdk-go
```

### Quick Start (Go)

```go
package main

import (
	"fmt"
	"log"
	"github.com/iotaledger/iota-sdk-go/iota_sdk"
)

func main() {
	// Connect to devnet / testnet / mainnet
	client := iota_sdk.GraphQlClientNewTestnet()
	// Or: iota_sdk.GraphQlClientNewMainnet()
	// Or: iota_sdk.GraphQlClientNew("https://your-endpoint.com")

	chainID, err := client.ChainId()
	if err.(*iota_sdk.SdkFfiError) != nil {
		log.Fatalf("Failed to get chain ID: %v", err)
	}
	fmt.Println("Chain ID:", chainID)
}
```

Available examples: chain info, coin balances, transactions, address/key management.

**Source:** [github.com/iotaledger/iota-sdk-go](https://github.com/iotaledger/iota-sdk-go)

## IOTA Gas Station (Sponsored Transactions)

A service that enables **gasless transactions** — users don't need IOTA tokens to pay for gas. The gas station manages a pool of gas coins and co-signs transactions.

**Source:** [github.com/iotaledger/gas-station](https://github.com/iotaledger/gas-station)
**Docs:** [docs.iota.org/operator/gas-station/](https://docs.iota.org/operator/gas-station/)

### How It Works

1. User builds a transaction
2. App calls Gas Station API to **reserve gas** (`POST /v1/reserve_gas`)
3. User signs the transaction locally
4. App submits signed tx + signature to Gas Station (`POST /v1/execute_tx`)
5. Gas Station co-signs and broadcasts on-chain

### TypeScript Example (Sponsored Transaction)

```typescript
import { IotaClient } from '@iota/iota-sdk/client';
import { Transaction } from '@iota/iota-sdk/transactions';
import { toB64 } from '@iota/bcs';
import axios from 'axios';

const client = new IotaClient({ url: nodeUrl });
const tx = new Transaction();

// Build your transaction (e.g., a Move call)
tx.moveCall({
  target: `0x2::clock::timestamp_ms`,
  arguments: [tx.object('0x6')],
});

// 1. Reserve gas from the gas station
const gasBudget = 50_000_000;
const reservation = await axios.post(gasStationUrl + '/v1/reserve_gas', {
  gas_budget: gasBudget,
  reserve_duration_secs: 10,
}, { headers: { Authorization: `Bearer ${gasStationToken}` } });

const { sponsor_address, reservation_id, gas_coins } = reservation.data.result;

// 2. Set sponsor details on the transaction
tx.setSender(senderAddress);
tx.setGasOwner(sponsor_address);
tx.setGasPayment(gas_coins);
tx.setGasBudget(gasBudget);

// 3. User signs locally
const unsignedTxBytes = await tx.build({ client });
const signedTx = await keypair.signTransaction(unsignedTxBytes);

// 4. Submit to gas station for co-signing + execution
const result = await axios.post(gasStationUrl + '/v1/execute_tx', {
  reservation_id,
  tx_bytes: toB64(unsignedTxBytes),
  user_sig: signedTx.signature,
});
console.log('Tx digest:', result.data.effects.transactionDigest);
```

### Running the Gas Station (Docker)

```bash
git clone https://github.com/iotaledger/gas-station
cd gas-station/docker
../utils/./gas-station-tool.sh generate-sample-config --config-path config.yaml --docker-compose -n testnet
GAS_STATION_AUTH=your-token docker-compose up
```

### Configuration (config.yaml)

```yaml
signer-config:
  local:
    keypair: AKT1Ghtd+yNbI9fFCQin3FpiGx8xoUdJMe7iAhoFUm4f  # base64
rpc-host-ip: 0.0.0.0
rpc-port: 9527
storage-config:
  redis:
    redis_url: "redis://127.0.0.1"
fullnode-url: "https://api.testnet.iota.cafe"
coin-init-config:
  target-init-balance: 100000000
  refresh-interval-sec: 86400
daily-gas-usage-cap: 1500000000000
```

## IOTA Names (Naming Service)

On-chain naming service for IOTA — register human-readable `.iota` names that map to addresses.

**Source:** [github.com/iotaledger/iota-names](https://github.com/iotaledger/iota-names)

### Register a Name (CLI)

```bash
# Requires IOTA CLI built with iota-names feature
cargo install --locked --bin iota --features=iota-names --path crates/iota \
  --git https://github.com/iotaledger/iota.git

# Register
iota name register myname.iota
```

### Local Development Setup

```bash
iota start --force-regenesis --with-faucet
iota client new-env --alias localnet --rpc http://127.0.0.1:9000 --graphql http://127.0.0.1:9125
iota client switch --env localnet
iota client faucet

# Deploy IOTA Names contracts
cd iota-names/scripts && pnpm ts-node init/init.ts localnet
```

## IOTA Multisig Manager ("Sagat")

A full-stack platform for managing IOTA multisig wallets. Web UI + API for creating and managing multi-signature accounts.

**Source:** [github.com/iotaledger/iota-multisig-manager](https://github.com/iotaledger/iota-multisig-manager)

### Architecture

- **Backend:** Bun + Hono + PostgreSQL + Drizzle ORM
- **Frontend:** React + Vite + TypeScript + Tailwind CSS
- **Blockchain:** IOTA Network via `@iota/iota-sdk`

### Quick Start

```bash
git clone https://github.com/iotaledger/iota-multisig-manager
cd iota-multisig-manager
cp api/.env.test api/.env
docker compose up postgres -d
bun install
(cd api && bun run db:migrate)
bun run dev
```

## References

- [CLI Reference](https://docs.iota.org/developer/references/cli)
- [TypeScript SDK](https://docs.iota.org/developer/ts-sdk/)
- [Rust SDK](https://docs.iota.org/developer/references/rust-sdk) | [docs.rs/iota-sdk](https://docs.rs/iota-sdk)
- [Go SDK](https://github.com/iotaledger/iota-sdk-go)
- [GraphQL RPC](https://docs.iota.org/developer/getting-started/graphql-rpc)
- [Gas Station Docs](https://docs.iota.org/operator/gas-station/)
- [Gas Station API Reference](https://docs.iota.org/operator/gas-station/api-reference/)
- [IOTA Names](https://github.com/iotaledger/iota-names)
- [Multisig Manager](https://github.com/iotaledger/iota-multisig-manager)
