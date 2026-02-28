# IOTA Identity — Decentralized Identity (DID & Verifiable Credentials)

## Overview

IOTA Identity is a Rust implementation of **Self-Sovereign Identity (SSI)** on IOTA Rebased. It implements:

- **W3C Decentralized Identifiers (DID)** — [did-core spec](https://www.w3.org/TR/did-core/)
- **W3C Verifiable Credentials** — [vc-data-model spec](https://www.w3.org/TR/vc-data-model/)
- **IOTA DID Method** — DIDs anchored on IOTA Rebased ledger

Key features: secure key storage (pluggable KMS), domain linkage, selective disclosure (SD-JWT), zero-knowledge proofs (BBS+), post-quantum signatures (PQ/T hybrid).

**Source:** [github.com/iotaledger/identity.rs](https://github.com/iotaledger/identity.rs) (main branch = IOTA Rebased compatible)
**Docs:** [docs.iota.org/iota-identity](https://docs.iota.org/iota-identity)

## Bindings

- **Rust** — Native implementation
- **WASM/TypeScript/JavaScript** — via [bindings/wasm](https://github.com/iotaledger/identity.rs/tree/main/bindings/wasm/identity_wasm/)
- **gRPC** — Experimental [gRPC services](https://github.com/iotaledger/identity.rs/tree/main/bindings/grpc/)

## Example: Create a DID Document (Rust)

```rust
use examples::create_did_document;
use examples::get_funded_client;
use examples::get_memstorage;

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    // Create client and get funded account
    let storage = get_memstorage()?;
    let identity_client = get_funded_client(&storage).await?;

    // Create and publish DID document on-chain
    let (document, _) = create_did_document(&identity_client, &storage).await?;
    println!("Published DID document: {document:#}");

    // Resolve it back from chain
    let resolved = identity_client.resolve_did(document.id()).await?;
    println!("Resolved DID document: {resolved:#}");

    Ok(())
}
```

## Example: Create & Verify a Verifiable Credential (Rust)

```rust
use identity_eddsa_verifier::EdDSAJwsVerifier;
use identity_iota::core::{json, FromJson, Object, Url};
use identity_iota::credential::{
    Credential, CredentialBuilder, DecodedJwtCredential, FailFast,
    Jwt, JwtCredentialValidationOptions, JwtCredentialValidator, Subject,
};
use identity_iota::did::DID;
use identity_iota::storage::{JwkDocumentExt, JwsSignatureOptions};

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    // Create issuer with DID
    let issuer_storage = get_memstorage()?;
    let issuer_client = get_funded_client(&issuer_storage).await?;
    let (issuer_doc, issuer_vm) = create_did_document(&issuer_client, &issuer_storage).await?;

    // Create holder with DID
    let holder_storage = get_memstorage()?;
    let holder_client = get_funded_client(&holder_storage).await?;
    let (holder_doc, _) = create_did_document(&holder_client, &holder_storage).await?;

    // Build credential subject
    let subject: Subject = Subject::from_json_value(json!({
        "id": holder_doc.id().as_str(),
        "name": "Alice",
        "degree": {
            "type": "BachelorDegree",
            "name": "Bachelor of Science and Arts",
        },
        "GPA": "4.0",
    }))?;

    // Build and sign the credential as JWT
    let credential: Credential = CredentialBuilder::default()
        .id(Url::parse("https://example.edu/credentials/3732")?)
        .issuer(Url::parse(issuer_doc.id().as_str())?)
        .type_("UniversityDegreeCredential")
        .subject(subject)
        .build()?;

    let credential_jwt: Jwt = issuer_doc
        .create_credential_jwt(
            &credential, &issuer_storage, &issuer_vm,
            &JwsSignatureOptions::default(), None,
        )
        .await?;

    // Validate the credential
    let decoded: DecodedJwtCredential<Object> =
        JwtCredentialValidator::with_signature_verifier(EdDSAJwsVerifier::default())
            .validate::<_, Object>(
                &credential_jwt, &issuer_doc,
                &JwtCredentialValidationOptions::default(),
                FailFast::FirstError,
            )
            .unwrap();

    println!("VC validated: {:#}", decoded.credential);
    Ok(())
}
```

## Available Examples

### Basic (CRUD)

| Example | Description |
|---------|-------------|
| `0_create_did` | Create and publish a DID Document |
| `1_update_did` | Update a DID document |
| `2_resolve_did` | Resolve an existing DID |
| `3_deactivate_did` | Deactivate a DID |
| `5_create_vc` | Create and verify Verifiable Credentials |
| `6_create_vp` | Create and verify Verifiable Presentations |
| `7_revoke_vc` | Revoke a Verifiable Credential |
| `8_legacy_stronghold` | Use Stronghold for secure storage |

### Advanced

| Example | Description |
|---------|-------------|
| `4_identity_history` | Fetch identity history |
| `5_custom_resolution` | Custom DID resolution handlers |
| `6_domain_linkage` | Link a domain to a DID and verify |
| `7_sd_jwt` | Selective disclosure VCs (SD-JWT) |
| `8_status_list_2021` | Revocation via StatusList2021 |
| `9_zkp` | Anonymous Credentials with BBS+ |
| `10_zkp_revocation` | ZKP credential revocation |
| `11_linked_verifiable_presentation` | Link public VP to identity |
| `12_pq` | Post-quantum signature VCs |
| `13_hybrid` | PQ/T hybrid signature VCs |

## Running Examples

```bash
# Against testnet (no IOTA_IDENTITY_PKG_ID needed for official networks)
API_ENDPOINT=https://api.testnet.iota.cafe cargo run --release --example 0_create_did

# Against local network (need to deploy identity package first)
IOTA_IDENTITY_PKG_ID=0x222741bb... cargo run --release --example 0_create_did
```

## Enterprise Use Cases

IOTA Identity is central to the **IOTA Trust Framework** used by:
- **TWIN** — Trade document verification across supply chains
- **DPP Demonstrator** — Digital Product Passport with verifiable manufacturer/repairer identities
- **ADAPT** — African trade corridor identity verification
- **Hierarchies** — Trust delegation chains (government → manufacturer → repairer)

## References

- [IOTA Identity Docs](https://docs.iota.org/iota-identity)
- [Rust API Reference](https://iotaledger.github.io/identity/identity_iota/index.html)
- [WASM API Reference](https://docs.iota.org/references/iota-identity/wasm/api_ref)
- [IOTA DID Method Spec](https://docs.iota.org/references/iota-identity/iota-did-method-spec/)
- [Rust Examples](https://github.com/iotaledger/identity.rs/tree/main/examples)
- [WASM Examples](https://github.com/iotaledger/identity.rs/tree/main/bindings/wasm/identity_wasm/examples)
- [Source Code](https://github.com/iotaledger/identity.rs)
