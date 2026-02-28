# TWIN — Trade Worldwide Information Network

## Table of Contents

- [Overview](#overview)
- [1. What is TWIN?](#1-what-is-twin)
- [2. Architecture](#2-architecture)
- [3. TWIN and IOTA Rebased](#3-twin-and-iota-rebased)
- [4. Real-World Deployments](#4-real-world-deployments)
- [5. For Developers](#5-for-developers)
- [6. TWIN Foundation](#6-twin-foundation)
- [7. ADAPT Initiative](#7-adapt-initiative)
- [8. Resources and Links](#8-resources-and-links)

## Overview

TWIN (Trade Worldwide Information Network) is an open-source data pipeline for global digital trade. It connects businesses, governments, and organizations to share data and documents across global value chains — powered by IOTA Rebased as its verification and immutability layer.

TWIN represents one of the most significant real-world use cases for IOTA, with **live trade transactions anchored on IOTA Mainnet since January 2026**.

**Key facts:**
- Targets the **$35+ trillion** global trade market
- Governed by the **TWIN Foundation** (6 founding organizations)
- Backed by **£3.5 million** UK Government Freeport seed capital for the Digital Trade Testbed
- Delivered trade data to UK authorities **up to 20 hours earlier** than existing processes
- Built on 4+ years of R&D by the IOTA Foundation

## 1. What is TWIN?

TWIN is an open, interoperable software platform for **data integrity and self-sovereign data management** in global trade. It is:

- **A digital pipeline** connecting all parties in international trade and supply chains
- **Built on IOTA** (public, permissionless DLT) for trust and immutability
- **Standards-based** — W3C, UN/CEFACT, Gaia-X, IDSA compliant
- **Open-source** — full code and specifications publicly available

### What Problems Does TWIN Solve?

| Problem | TWIN Solution |
|---------|---------------|
| Late supply chain data | Digital pipeline delivers data hours/days earlier |
| Manual document handling | Automated, verified document exchange |
| Vendor lock-in | Open-source, interoperable by design |
| Lack of trust between parties | Verifiable credentials + DLT anchoring |
| Data sovereignty concerns | Organizations maintain full control over their data |
| SME exclusion from digital trade | Democratized access via open APIs |

### Key Benefits

- **Interoperability by Design** — works with existing systems without vendor lock-in
- **Data Sovereignty** — organizations maintain full control over their data
- **Scalability & Security** — built on IOTA blockchain for trust and transparency
- **Open Standards** — W3C, UN/CEFACT, and other global standards
- **Democratized Access** — enables SMEs to participate in digital trade

## 2. Architecture

TWIN uses a layered architecture as described in the [TWIN Whitepaper (V1.0, January 2026)](https://twindev.org/docs/twin-white-paper):

### Three-Layer Architecture

```
┌─────────────────────────────────────────┐
│         APPLICATION LAYER               │
│  Digital Product Passports, Trade       │
│  Finance, Customs, Supply Chain UIs     │
├─────────────────────────────────────────┤
│      DATA & SERVICES LAYER              │
│  Verifiable Credentials, DIDs,          │
│  Data Rights (W3C ODRL),               │
│  Federated Catalogues, Registries       │
├─────────────────────────────────────────┤
│       INFRASTRUCTURE LAYER              │
│  IOTA DLT (Mainnet), TWIN Nodes,       │
│  Decentralized Storage, APIs            │
└─────────────────────────────────────────┘
```

### Core Components

- **TWIN Nodes** — Federated nodes that synchronize catalogues and registries
- **IOTA DLT Layer** — IOTA Mainnet for immutable anchoring of trade documents and audit trails
- **Decentralized Identifiers (DIDs)** — W3C-standard digital identities via IOTA Identity
- **Verifiable Credentials** — Cryptographically signed attestations (certifications, licenses, etc.)
- **Data Space Connector** — Secure, modular data exchange respecting data sovereignty
- **Data Rights Management** — Based on W3C ODRL framework (Policy Administration/Decision Points)

### How Data Flows

Using the example of a poultry export from Poland to the UK:

1. **Producer** creates digital records (origin, processing, health checks)
2. **Freight forwarder** collects documents, shares via TWIN APIs
3. **TWIN pipeline** consolidates data: commercial docs, certificates, dispatch data, temperature monitoring
4. **Key documents** are digitally signed and anchored on IOTA Mainnet
5. **UK Government** receives verified supply chain data via Information Sharing Network
6. **Border authorities** get granular, verified information 20+ hours earlier than existing processes

## 3. TWIN and IOTA Rebased

TWIN uses IOTA Rebased as its **trust and verification layer**:

| IOTA Feature | How TWIN Uses It |
|-------------|-----------------|
| **Move smart contracts** | Automate trade finance, compliance checks |
| **IOTA Identity** | Decentralized Identifiers (DIDs) for businesses, governments |
| **IOTA Notarization** | Immutable audit trails for trade documents |
| **IOTA Hierarchies** | Trust delegation across organizations, federations |
| **IOTA Gas Station** | Sponsored transactions for seamless UX (no gas for trade users) |
| **Object model** | Trade documents, certificates as on-chain objects |
| **Low fees** | ~0.005 IOTA per tx — viable for high-volume trade data |

### Live on Mainnet

As of **January 2026**, live trade transactions processed via TWIN are anchored on the IOTA Mainnet. This represents sustained, non-speculative network usage driven by real economic activity.

## 4. Real-World Deployments

### UK Digital Trade Testbed

The highest-profile TWIN deployment:

- **Location:** Teesside, UK (seaport and airport)
- **Partners:** IOTA Foundation + Teesside University
- **Funding:** £3.5 million in UK Government Freeport seed capital
- **Government involvement:** 4 UK Government trade officials seconded to IOTA Foundation for 12 months
- **Results from 2025 trials:** Supply chain data reached UK authorities up to **20 hours earlier**

The UK testbed combines TWIN with physical solutions (autonomous vehicles, geo-locating devices) for comprehensive trade digitalization.

**Economic impact potential** (per ICC UK and London School of Economics):
- £25 billion in trade growth
- £224 billion in efficiency savings
- 35% efficiency gain for SMEs
- 1.3% boost to UK GDP from full trade digitalization

### East Africa (TLIP)

TLIP (Trade Logistics Information Pipeline) is TWIN's application in East African trade:

- Integrated with **Kenya Ports Authority (KPA)**
- Reduced document availability from **weeks to minutes** for Kenyan exports to the UK
- API connectivity with Kilindini Waterfront Automated Terminal Operating System
- Covers: pre-advice, incoming, stacked-to-delivered statuses for exports; import processing

### GLEIF Partnership

- MoU with **GLEIF** (Global Legal Entity Identifier Foundation) signed in Q3 2025
- Links the global LEI system to TWIN's on-chain infrastructure
- Simplifies cross-border business authentication and compliance

### Salus — Trade Finance for Critical Minerals

- Tokenizes trade documents on IOTA
- Anchors audit trails for mineral supply chains
- Automates payments with smart contracts
- Addresses the $2.5 trillion trade finance gap

## 5. For Developers

### TWIN Developer Portal

The main developer resource: **[twindev.org](https://twindev.org/)**

Available resources:
- [Introduction](https://twindev.org/docs/intro) — overview of TWIN technology
- [Whitepaper](https://twindev.org/docs/twin-white-paper) — reference architecture (V1.0, Jan 2026)
- [Tutorials](https://twindev.org/docs/tutorials/twin-iota-dlt-identity-howto) — integration guides
- [Applications](https://twindev.org/docs/apps) — available applications
- [Packages](https://twindev.org/docs/pkgs) — software packages
- [Roadmap](https://twindev.org/docs/roadmap) — development progress
- [Open API](https://editor.swagger.io/?url=https://raw.githubusercontent.com/twinfoundation/node/next/apps/node/docs/open-api/spec.json) — TWIN Node REST API (Swagger)

### GitHub

TWIN source code and documentation: [github.com/twinfoundation](https://github.com/twinfoundation/docs)

### IOTA Products Used by TWIN

If you're building applications that integrate with TWIN, these are the IOTA tools you'll use:

| Product | Purpose | Docs |
|---------|---------|------|
| **IOTA Identity** | DIDs and Verifiable Credentials | [docs.iota.org/developer/iota-identity](https://docs.iota.org/developer/iota-identity/) |
| **IOTA Notarization** | Immutable document anchoring | [docs.iota.org/developer/iota-notarization](https://docs.iota.org/developer/iota-notarization/) |
| **IOTA Hierarchies** | Trust delegation networks | [docs.iota.org/developer/iota-hierarchies](https://docs.iota.org/developer/iota-hierarchies/) |
| **IOTA Gas Station** | Sponsored transactions | [docs.iota.org/operator/gas-station](https://docs.iota.org/operator/gas-station/) |

### Technical Webinars

TWIN hosts open technical webinars covering architecture and use cases:
- [TWIN Technical Webinar (YouTube)](https://www.youtube.com/watch?v=i-KkG9NsHJg) — covers Digital Product Passports, three-party document sharing

## 6. TWIN Foundation

The TWIN Foundation was officially launched on **May 8, 2025** in Lusaka, Zambia. It governs TWIN as a **global public good** for digital trade.

### Founding Organizations

1. **TradeMark Africa** — trade facilitation across Africa
2. **IOTA Foundation** — technology provider (blockchain, identity, smart contracts)
3. **World Economic Forum** — global policy and standards
4. **Tony Blair Institute for Global Change** — government advisory
5. **Chartered Institute of Export & International Trade** (UK) — trade standards
6. **Global Alliance for Trade Facilitation** — public-private trade partnerships

## 7. ADAPT Initiative

**ADAPT** (Africa Digital Trade) is a continental initiative to digitalize African trade, co-developed with the AfCFTA Secretariat:

- **Goal:** Double intra-African trade by 2035
- **Technology:** TWIN + IOTA as the infrastructure layer
- **2026 plans:** 3 countries confirmed for deployment, 5 more beginning pilot programs
- Integrates digital identity, cross-border data exchange, and interoperable finance
- Could unlock tens of billions in economic value

## 8. Resources and Links

### Official

| Resource | URL |
|----------|-----|
| TWIN Website | [twin.org](https://www.twin.org/home) |
| TWIN Developer Portal | [twindev.org](https://twindev.org/) |
| TWIN Whitepaper (PDF) | [V1.0 January 2026](https://twindev.org/pdf/TWIN_Whitepaper_Reference_Architecture-V1.0-January-2026.pdf) |
| TWIN GitHub | [github.com/twinfoundation](https://github.com/twinfoundation/docs) |
| TWIN Node API (Swagger) | [Open API Spec](https://editor.swagger.io/?url=https://raw.githubusercontent.com/twinfoundation/node/next/apps/node/docs/open-api/spec.json) |
| TLIP (East Africa) | [tlip.io](https://www.tlip.io/) |

### Blog Posts

- [UK Digital Trade Testbed](https://blog.iota.org/turning-pilots-into-progress/) — £3.5M Teesside partnership
- [TWIN Foundation Launch](https://blog.iota.org/twin-foundation-launched/) — May 2025, Lusaka
- [GLEIF Partnership](https://blog.iota.org/gleif-partnership/) — global business identity
- [Salus Trade Finance](https://blog.iota.org/trade-finance-reinvented/) — critical minerals
- [IOTA 2025 Review](https://blog.iota.org/iota-2025-review/) — TWIN progress summary
- [Q3 2025 Progress](https://blog.iota.org/q3-2025-progress-update/) — TWIN technical updates

### Related IOTA Skill Docs

- [ecosystem.md](ecosystem.md) — broader IOTA ecosystem
- [roadmap-starfish.md](roadmap-starfish.md) — IOTA protocol roadmap
- [architecture.md](architecture.md) — IOTA Rebased technical architecture
