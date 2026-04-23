# VPDOT — Verifiable Physical-to-Digital Ownership Transitions

<p align="center">
  <img src="https://img.shields.io/badge/Status-Research%20%2F%20Pre--Development-orange?style=flat-square"/>
  <img src="https://img.shields.io/badge/Fellowship-Ethereum%20Foundation%202026-627EEA?style=flat-square&logo=ethereum"/>
  <img src="https://img.shields.io/badge/Network-Optimism%20Sepolia-red?style=flat-square"/>
  <img src="https://img.shields.io/badge/Licence-MIT-green?style=flat-square"/>
  <img src="https://img.shields.io/badge/Language-Solidity%20%7C%20C%2B%2B-blue?style=flat-square"/>
</p>

> **Ethereum Foundation PhD Fellowship 2026 — Software-Defined Operations for Small Businesses**
>
> This repository is the research workspace for a PhD fellowship proposal.
> No production code has been deployed yet. Development begins upon fellowship approval.
> This README documents the planned architecture, scope, and research context.

---

## Table of Contents

- [Overview](#overview)
- [The Problem](#the-problem)
- [Proposed Solution](#proposed-solution)
- [Architecture](#architecture)
- [Hardware Stack](#hardware-stack)
- [Research Phases](#research-phases)
- [Repository Structure](#repository-structure)
- [Broader Applicability](#broader-applicability)
- [Prior Work](#prior-work)
- [Research Team](#research-team)
- [Licence](#licence)

---

## Overview

Pharmaceutical SMEs — cold-chain distributors, independent wholesalers, and dispensing
pharmacies — operate under some of the most demanding regulatory requirements in European
commerce, yet with digital infrastructure that is often fragile and fragmented.

**VPDOT** proposes an Ethereum-native coordination layer built around two complementary
innovations:

1. **Physical Unclonable Functions (PUFs)** — hardware-embedded, manufacturer-unique
   cryptographic identifiers that cannot be cloned or extracted — sign IoT sensor
   attestations at the device level, enabling genuinely oracle-free physical state
   verification on-chain without a trusted intermediary.

2. **A formal Solidity ownership state machine** that uses these authenticated attestations
   as autonomous triggers for conditional escrow release, insurance payouts, and
   multi-party compliance records — replacing weeks of manual reconciliation with
   deterministic on-chain logic.

---

## The Problem

When temperature-sensitive medicines are damaged in transit, establishing liability and
triggering insurance can take **weeks of manual reconciliation** across incompatible ERP
and SaaS systems. The EU Falsified Medicines Directive (2011/62/EU) mandates end-to-end
custody verification, yet most SME implementations depend on centralised national hubs
that reintroduces the very platform risk, vendor lock-in, and opaque reconciliation
Overhead, the regulation was designed to eliminate.

### Four Cost Dimensions VPDOT Targets

|            Cost Type                |                            Description                                          |
|-------------------------------------|---------------------------------------------------------------------------------|
| **Handoff ambiguity**               | Time and legal overhead when ownership is disputed at cold-chain handoff points |
| **Reconciliation overhead**         | Labour hours resolving discrepancies between ERP records, paper batch certificates, and EMVS hub data |
| **Insurance dispute latency**       | Average time from cold-chain excursion event to liability determination and payout|
| **Compliance documentation burden** | GDP-mandated audit trail costs borne disproportionately by small distributors     |

---

## Proposed Solution

### Solidity Ownership State Machine

```solidity
enum AssetState {
    Manufactured,   // ERC-721 token minted with PUF fingerprint + IPFS batch metadata
    InTransit,      // PUF-signed condition hashes logged at each handoff; streaming payment released
    Received,       // PUF attestation within temperature bounds; escrow released; compliance log emitted
    Disputed,       // Cold-chain excursion detected; transfers frozen; Kleros arbitration optionally invoked
    Recalled        // Regulator/Manufacturer emits on-chain recall signal
}
```

### PUF Attestation Protocol

```
ESP32 Boot
    │
    ▼
SRAM Power-Up State  ──►  PUF Key Derivation (no persistent storage)
    │
    ▼
Sensor Readings (temp, humidity, vibration, GPS)
    │
    ▼
Hash + Session Nonce  ──►  secp256k1 ECDSA Signature
    │
    ▼
HTTPS JSON-RPC  ──►  Optimism Sepolia L2
    │
    ▼
Solidity ecrecover validates:
    (a) Signature validity
    (b) Nonce freshness (replay prevention)
    (c) Registered device fingerprint match
    │
    ▼
State Transition Triggered  ──►  Escrow / Insurance / Compliance Log
```

### Why Oracle-Free Matters

| Approach | Trust Dependency | VPDOT Equivalent |
|----------|-----------------|-----------------|
| Chainlink oracle | Oracle node operators | None — PUF signs directly |
| Cloud-hosted MQTT broker | Cloud provider | None — ESP32 signs directly |
| Software key storage | Secure enclave / HSM vendor | None — SRAM entropy at runtime |

---

## Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                     ETHEREUM L2 (Optimism)                  │
│                                                             │
│  ┌──────────────────┐    ┌──────────────────────────────┐   │
│  │  VPDOTAsset.sol  │    │   ConditionalEscrow.sol      │   │
│  │  (ERC-721 +      │◄──►│   (USDC/DAI release on       │   │
│  │   AssetState SM) │    │    PUF-signed delivery)      │   │
│  └────────┬─────────┘    └──────────────────────────────┘   │
│           │                                                 │
│  ┌────────▼─────────┐    ┌──────────────────────────────┐   │
│  │ PUFVerifier.sol  │    │  InsuranceTrigger.sol        │   │
│  │ (ecrecover +     │    │  (automated payout on        │   │
│  │  device registry)│    │   excursion attestation)     │   │
│  └──────────────────┘    └──────────────────────────────┘   │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  OpenZeppelin AccessControl                          │   │
│  │  Roles: Manufacturer | Carrier | Pharmacist          │   │
│  │         | Insurer | Regulator                        │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
              ▲                          ▲
              │ HTTPS JSON-RPC           │ Kleros Arbitration
              │                          │ (Disputed state)
┌─────────────┴──────────┐
│    ESP32 IoT Node      │
│                        │
│  SRAM-PUF Key Deriv.   │
│  DHT22  (temp/humid)   │
│  ADXL345 (vibration)   │
│  GPS    (location)     │
│  secp256k1 ECDSA Sign  │
│                        │
│  Target BoM: ~€28/node │
└────────────────────────┘
```
---

## Hardware Stack

| Component            |          Specification          |             Purpose               |
|----------------------|---------------------------------|-----------------------------------|
| **ESP32**            | Dual-core 240MHz, 520KB SRAM    | Main MCU; SRAM-PUF entropy source |
| **DHT22**            | ±0.5°C accuracy, −40°C to +80°C | Temperature & humidity            |
| **ADXL345**          | ±16g range, 13-bit resolution   | Vibration & shock detection       |
| **GPS module**       | NMEA 0183, 2.5m CEP             | Location attestation              |
| **Total BoM target** |                   —             | **~€28 per node**                 |

SRAM-PUF reliability: consistent key derivation demonstrated at −25°C, and +25°C,
directly relevant to pharmaceutical cold-chain environments.

---

## Research Phases

| Milestone | Period | Key Tasks | Deliverables | Budget |
|-----------|--------|-----------|--------------|--------|
| **M1** — Operational Mapping | Months 1–3 | 10–15 SME interviews (Italy); BPMN process mapping; EU GDP/GDPR regulatory analysis; adoption barrier synthesis | Anonymised interview dataset; BPMN maps; coordination-failure taxonomy; regulatory compatibility report | $6,000 |
| **M2** — Primitive Design | Months 4–6 | Formal Solidity specification; SRAM-PUF + ECDSA verifier; escrow & insurance library; gas profiling on Optimism/Arbitrum | Solidity codebase v1.0; PUF protocol spec; security & gas analysis report; solution benchmark rubric | $6,000 |
| **M3** — Hardware Prototype & Pilot | Months 7–10 | ESP32/PUF prototype build; two pilot scenarios on Optimism Sepolia; ≥2 SME usability sessions; workflow comparison | Live hardware prototype; deployed testnet pilot; pilot evaluation dataset; open-source repository v2.0 | $7,500 |
| **M4** — Dissemination | Months 11–12 | Peer-reviewed manuscript; VPI ERC/EIP draft; final repository documentation; Devcon/ETHGlobal presentation | Submitted paper; EIP draft; final open-source release; conference presentation | $4,500 |
| **Total** | **12 months** | **~960 research hours** | | **$24,000** |

---

## Repository Structure

```
VPDOT/
├── contracts/                          # Solidity smart contracts
│   ├── VPDOTAsset.sol                  # ERC-721 + AssetState state machine
│   ├── PUFVerifier.sol                 # ecrecover + device registry
│   ├── ConditionalEscrow.sol           # PUF-gated payment release
│   ├── InsuranceTrigger.sol            # Automated excursion payouts
│   ├── TripleEntryLedger.sol           # On-chain GDP-compliant audit log
│   └── interfaces/
│       └── IVerifiablePhysicalAsset.sol  # Draft VPI ERC interface
│
├── firmware/                           # ESP32 firmware (C++)
│   ├── puf/                            # SRAM-PUF key derivation
│   ├── sensors/                        # DHT22, ADXL345, GPS drivers
│   └── attestation/                    # ECDSA signing + L2 transmission
│
├── scripts/                            # Hardhat deployment & gas profiling
│   ├── deploy.js
│   └── gasProfile.js
│
├── test/                               # Contract test suite
│   ├── VPDOTAsset.test.js
│   ├── PUFVerifier.test.js
│   └── ConditionalEscrow.test.js
│
├── docs/                               # Research documentation
│   ├── architecture.md                 # System design detail
│   ├── puf-protocol.md                 # PUF attestation specification
│   ├── regulatory-analysis.md          # EU FMD / GDP / GDPR analysis
│   ├── adoption-barrier-framework.md   # Multi-disciplinary barrier taxonomy
│   ├── solution-evaluation.md          # Safe/Aragon/Colony/Hats/OZG/Kleros rubric
│   └── hardware-bom.md                 # Bill of materials & assembly guide
│
├── pilot/                              # SME pilot data & reports
│   ├── scenarios/
│   │   ├── clean-delivery.md
│   │   └── cold-chain-excursion.md
│   └── evaluation/
│
├── .gitignore
├── LICENSE
├── hardhat.config.js                   # (to be added in M2)
└── README.md
```

---

## Broader Applicability

While VPDOT targets pharmaceutical SMEs as its primary validation domain — chosen for
its regulatory stringency, liability stakes, and cold-chain complexity — the core
primitives are domain-agnostic.

Prior work by the research group has already demonstrated a live blockchain-IoT
monitoring pipeline in an operational agrifood SME setting, validating transferability to:

| Vertical | Application |
|----------|-------------|
|    Agrifood distribution    | Temperature-authenticated produce custody |
|    Restaurant supply chains | Ingredient provenance and freshness attestation |
|    Fresh produce cold-chain | Automated liability on excursion events |
|    Any regulated physical commerce | Generalised via the VPI ERC standard |

The VPI (Verifiable Physical Identity) ERC standard being developed as part of this
project is designed from the outset to be vertical-agnostic, providing a standard
interface for PUF-anchored physical asset authentication on Ethereum.

---

## Prior Work

| Publication | Venue | Link | Relevance to VPDOT |
|-------------|-------|------|-------------------|
| A Move Sui library for secure, certified and trusted supply chain ownership management | IEEE/ACM WETSEB 2025 | [IEEE Xplore](https://ieeexplore.ieee.org/document/11039281) | Direct ancestor: three-state ownership model migrated and extended to Ethereum with PUF authentication |
| Real-Time Monitoring and Transparency in Pizza Production Using IoT and Blockchain | arXiv 2025 | [arXiv:2507.02536](https://arxiv.org/abs/2507.02536) | Live agrifood SME deployment validating hardware-software approach and domain transferability |
| Comparative Evaluation of Blockchain Technologies for IoT Energy Monitoring in Residential Settings | DLT2025, CEUR-WS Vol. 4105 | [CEUR-WS](https://ceur-ws.org/Vol-4105/paper13.pdf) | Ethereum vs IOTA/Polygon/Hyperledger for IoT pipelines; scalability, cost, and latency benchmarks |

---

## Research Team

**Azmat Ullah** — PhD Candidate  
University of Cagliari & University of Camerino, Italy  
📧 azmat.ullah@unicam.it  
🔗 [ORCID](https://orcid.org/0009-0006-4452-3948) · [ResearchGate](https://www.researchgate.net/profile/Azmat-Ullah-26) · [GitHub](https://github.com/azmat1177)

**Supervisor: Prof. Roberto Tonelli**  
Department of Mathematics & Information Technology, University of Cagliari  
🔗 [Academic Profile](https://web.unica.it/unica/page/it/roberto_tonelli)

---

## Licence

All code and documentation in this repository will be released under the **MIT Licence**
upon completion. See [LICENSE](./LICENSE) for details.

---

<p align="center">
  <sub>Submitted for consideration under the Ethereum Foundation PhD Fellowship Initiative 2026</sub>
</p>
