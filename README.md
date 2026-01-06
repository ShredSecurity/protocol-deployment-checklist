# Protocol Deployment Checklist

**Version:** v1.0.0  
**Status:** Stable  
**Last updated:** 2026-01-06

## Table of Contents

- [**Introduction**](#introduction)
- [**How to use this checklist (Normative Rules)**](#how-to-use-this-checklist-normative-rules)
- [**1. Pre-Deployment (Design & Review)**](#1-pre-deployment-design--review)
    - [1.1 Architecture & Upgradeability](#11-architecture--upgradeability)
    - [1.2 Threat Model & Deployment Risks](#12-threat-model--deployment-risks)
    - [1.3 Audits & Testing Readiness](#13-audits--testing-readiness)
    - [1.4 External Asset & Token Behavior Validation](#14-external-asset--token-behavior-validation)
- [**2. Deployment Plan (Scripts + Human Steps)**](#2-deployment-plan-scripts--human-steps)
    - [2.1 Deployment script sanity](#21-deployment-script-sanity)
    - [2.2 Proxy Deployment & Initialization Controls](#22-proxy-deployment--initialization-controls)
    - [2.3 Explorer Verification & Alias Consistency](#23-explorer-verification--alias-consistency)
    - [2.4 Admin & Key Management Requirements](#24-admin--key-management-requirements)
- [**3. Post-deployment checks (right after mainnet deploy)**](#3-post-deployment-checks-right-after-mainnet-deploy)
    - [3.1 On-Chain Read-Only Sanity Validation](#31-on-chain-read-only-sanity-validation)
    - [3.2 Functional Smoke Tests](#32-functional-smoke-tests)
    - [3.3 Monitoring, Alerting & Observability Setup](#33-monitoring-alerting--observability-setup)
    - [3.4 Emergency Procedures & Incident Response](#34-emergency-procedures--incident-response)

## Introduction

This checklist defines a baseline standard for deploying smart contract protocols to mainnet and other live networks.

It is intended to be used by protocol engineering teams during deployment and by auditors to verify that deployment, upgradeability, role configuration, and operational safeguards are correctly implemented prior to launch.

## How to use this checklist (Normative Rules)

- **Required?** → mark **Yes / No / N/A**
- N/A is permitted **only when** the condition in “**Applies When**” is not satisfied and must be justified in the Evidence/Note column.
- Each checklist item includes a Severity level that defines its impact on launch readiness.
- All resolved items should include **verifiable evidence** (transaction hash, explorer link, commit hash, or documentation reference).

### Severity Definition

- **BLOCKER**
	- A requirement that **must be satisfied before deployment to a live network**.
	- Any unresolved BLOCKER **prevents launch** and requires remediation or an explicit decision to delay deployment.
- **WARNING**
	- A requirement that **does not strictly block deployment**, but represents a **material security, operational, or reliability risk** if unmet.
	- WARNING items may proceed to launch **only if the risk is explicitly acknowledged, documented, and accepted** by the protocol’s responsible authority (e.g., multisig, DAO, or security lead).
- **INFO**
	- A **best-practice or observability recommendation** that improves safety, transparency, or operational maturity but **does not impact launch eligibility**.
	- INFO items should be addressed where practical and tracked for future improvements.

## 1. Pre-deployment (design & review)

### 1\.1 Architecture & upgradeability

| Check | Applies When | Severity | Required? | Evidence/Note |
| --- | --- | --- | --- | --- |
| Upgradeability pattern is explicitly documented | Protocol is upgradeable | BLOCKER |     |     |
| Each proxy has exactly one clearly defined admin | Proxies are used | BLOCKER |     |     |
| Proxy admin is not an EOA | Proxies are used | BLOCKER |     |     |
| Initialization functions are one-time only | Proxies are used | BLOCKER |     |     |
| Re-initialization is impossible | Proxies are used | BLOCKER |     |     |
| All privileged roles are explicitly listed | Any privileged access exists | WARNING |     |     |
| Capabilities of each privileged role are documented | Any privileged access exists | WARNING |     |     |
| Deterministic deployment assumptions documented | CREATE2 / CREATE3 used | WARNING |     |     |
| CREATE2 / CREATE3 salts are finalized | CREATE2 / CREATE3 used | BLOCKER |     |     |
| Deterministic addresses precomputed | CREATE2 / CREATE3 used | BLOCKER |     |     |
| Deterministic addresses consistent across chains | Multi-chain + deterministic deploy | BLOCKER |     |     |
| Deterministic deployer is trusted and verified | CREATE2 / CREATE3 used | BLOCKER |     |     |

### 1\.2 Threat Model & Deployment Risks

| Check | Applies When | Severity | Required? | Evidence/Note |
| --- | --- | --- | --- | --- |
| Deployment-specific threat model completed | Always | BLOCKER |     |     |
| MEV / frontrunning risks evaluated | Public deployment | BLOCKER |     |     |
| Deployment simulated against MEV scenarios | Public deployment | WARNING |     |     |
| Deployment txs simulated in Tenderly / Foundry | Always | WARNING |     |     |
| L2 sequencer risks evaluated | Deploying on L2 | BLOCKER |     |     |
| Critical steps separated into controlled txs | Admin / init actions exist | BLOCKER |     |     |
| Recovery plan for failed deployment exists | Always | BLOCKER |     |     |
| Recovery plan for failed initialization exists | Proxies are used | BLOCKER |     |     |

### 1\.3 Audits & Testing Readiness

| Check | Applies When | Severity | Required? | Evidence/Note |
| --- | --- | --- | --- | --- |
| Contracts audited by independent party | Always | BLOCKER |     |     |
| Proxy wiring included in audit | Proxies are used | BLOCKER |     |     |
| Fork-based mainnet tests executed | Always | WARNING |     |     |
| Unauthorized / double-init tests exist | Proxies are used | BLOCKER |     |     |
| Emergency pause tested | Pause functionality exists | BLOCKER |     |     |
| Storage layout compatibility tested | Proxies are used | BLOCKER |     |     |
| Upgrade invariants tested | Proxies are used | WARNING |     |     |
| Fuzz / Invariant Testing / Formal Verification executed | Complex logic | WARNING |     |     |

### 1.4 External Asset & Token Behavior Validation

| Check | Applies When | Severity | Required? | Evidence/Note |
| --- | --- | --- | --- | --- |
| External tokens listed per network | External assets used | BLOCKER |     |     |
| Token addresses verified | External assets used | BLOCKER |     |     |
| Token symbols verified | External assets used | WARNING |     |     |
| Token decimals verified | External assets used | BLOCKER |     |     |
| Token behavior assumptions documented | External assets used | BLOCKER |     |     |
| Wrapped native behavior verified | Native wrap assumed | BLOCKER |     |     |
| Native wrap assumptions documented | Native wrap assumed | BLOCKER |     |     |
| Canonical token metadata cross-referenced | Canonical source exists | INFO |     |     |
| Non-standard behavior documented | Non-standard tokens used | BLOCKER |     |     |

## 2. Deployment plan (scripts + human steps)

### 2.1 Deployment script sanity

| Check | Applies When | Severity | Required? | Evidence/Note |
| --- | --- | --- | --- | --- |
| Network and chain ID correct | Always | BLOCKER |     |     |
| RPC and gas configuration verified | Always | BLOCKER |     |     |
| Deterministic target addresses precomputed | CREATE2 / CREATE3 used | BLOCKER |     |     |
| Salts and factories verified | CREATE2 / CREATE3 used | BLOCKER |     |     |
| Script outputs implementation addresses | Always | BLOCKER |     |     |
| Script outputs proxy addresses | Proxies are used | BLOCKER |     |     |
| Script outputs admin addresses | Proxies are used | BLOCKER |     |     |
| Script outputs initialization parameters | Proxies are used | BLOCKER |     |     |
| No proxy deployed without admin | Proxies are used | BLOCKER |     |     |
| Deployer EOA cannot become admin unintentionally | Proxies are used | BLOCKER |     |     |

### 2.2 Proxy Deployment & Initialization Controls

| Check | Applies When | Severity | Required? | Evidence/Note |
| --- | --- | --- | --- | --- |
| Admin set before critical actions | Proxies are used | BLOCKER |     |     |
| Initialization executed via controlled tx | Proxies are used | BLOCKER |     |     |
| Initialization not publicly callable | Proxies are used | BLOCKER |     |     |
| Initialization not batched with third-party calls | Proxies are used | WARNING |     |     |
| Proxy admin verified on-chain | Proxies are used | BLOCKER |     |     |
| Proxy implementation verified on-chain | Proxies are used | BLOCKER |     |     |
| Initialization state verified | Proxies are used | BLOCKER |     |     |
| Initialization cannot be reset | Proxies are used | BLOCKER |     |     |
| Multi-chain deployment sequencing documented | Multi-chain deploy | BLOCKER |     |     |
| Per-chain deployment order verified | Multi-chain deploy | BLOCKER |     |     |

### 2.3 Explorer Verification & Alias Consistency

| Check | Applies When | Severity | Required? | Evidence/Note |
| --- | --- | --- | --- | --- |
| Implementation contracts verified | Always | BLOCKER |     |     |
| Proxies correctly labeled | Proxies are used | BLOCKER |     |     |
| Explorer implementation matches script output | Proxies are used | BLOCKER |     |     |
| Verified source matches audited commit | Always | BLOCKER |     |     |
| Bytecode matches audited build | Always | BLOCKER |     |     |
| Creation metadata cross-checked | Explorer API available | WARNING |     |     |
| Verification completed on all chains | Multi-chain deploy | BLOCKER |     |     |

### 2.4 Admin & Key Management Requirements

| Check | Applies When | Severity | Required? | Evidence/Note |
| --- | --- | --- | --- | --- |
| Admin roles held by multisig / DAO | Privileged admin exists | BLOCKER |     |     |
| Multisig signers validated | Multisig used | BLOCKER |     |     |
| Secure key storage in use | Privileged keys exist | BLOCKER |     |     |
| Secure backups exist | Privileged keys exist | WARNING |     |     |
| Role assignments executed and recorded | RBAC used | BLOCKER |     |     |
| Multisig quorum documented | Multisig used | BLOCKER |     |     |
| Admin rotation procedures documented | Multisig used | WARNING |     |     |
| Role transfers tested in staging | RBAC used | WARNING |     |     |

## 3. Post-deployment checks (right after mainnet deploy)

### 3.1 On-Chain Read-Only Sanity Validation

| Check | Applies When | Severity | Required? | Evidence/Note |
| --- | --- | --- | --- | --- |
| Proxy admin returns expected value | Proxies are used | BLOCKER |     |     |
| Proxy implementation returns expected value | Proxies are used | BLOCKER |     |     |
| Initialization / version variables correct | Proxies are used | BLOCKER |     |     |
| No unauthorized upgrade paths exist | Proxies are used | BLOCKER |     |     |
| No re-initialization paths exist | Proxies are used | BLOCKER |     |     |
| EIP-1967 slots inspected directly | Proxies are used | BLOCKER |     |     |
| State verified across multiple RPCs | Always | WARNING |     |     |

### 3.2 Functional smoke tests

| Check | Applies When | Severity | Required? | Evidence/Note |
| --- | --- | --- | --- | --- |
| Canary interaction executed | Always | BLOCKER |     |     |
| Core flows work with small values | Always | BLOCKER |     |     |
| Storage values sane | Always | WARNING |     |     |
| Cross-contract interactions validated | Multi-contract system | WARNING |     |     |
| Multi-chain edge cases tested | Multi-chain deploy | WARNING |     |     |

### 3.3 Monitoring, Alerting & Observability Setup

| Check | Applies When | Severity | Required? | Evidence/Note |
| --- | --- | --- | --- | --- |
| Alerts for upgrades configured | Proxies are used | BLOCKER |     |     |
| Alerts for admin / role changes | Privileged roles exist | BLOCKER |     |     |
| Economic anomaly alerts configured | Value-bearing protocol | WARNING |     |     |
| Dashboards live and ingesting | Analytics required | INFO |     |     |
| Indexers / subgraphs synced | Indexer used | BLOCKER |     |     |
| L2 risk monitoring enabled | Deploying on L2 | WARNING |     |     |

### 3.4 Emergency Procedures & Incident Response

| Check | Applies When | Severity | Required? | Evidence/Note |
| --- | --- | --- | --- | --- |
| Emergency pause confirmed functional | Pause exists | BLOCKER |     |     |
| Pauser role holders identified | Pause exists | BLOCKER |     |     |
| Incident response runbook documented | Always | BLOCKER |     |     |
| User communication plan documented | User-facing protocol | WARNING |     |     |
| Cross-chain emergency procedures defined | Multi-chain deploy | BLOCKER |     |     |
| Emergency drills scheduled | Always | INFO |     |     |


## About Shred Security

This checklist is authored and maintained by Shred Security, a research-driven security firm specializing in smart contract auditing, threat modeling, and blockchain security.

For review engagements, deployment war rooms, or full protocol audits, you can reach us at [Telegram](https://t.me/shredsecurity).

---

## License

© 2026 Shred Security

This work is licensed under the Creative Commons Attribution 4.0 International License (CC BY 4.0).

You are free to share and adapt this checklist for any purpose, including commercial use, provided appropriate credit is given to Shred Security.
