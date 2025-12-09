# Protocol Deployment Checklist

## Introduction

This checklist is a deployment-readiness framework designed by Shred Security for protocols, used by developers and auditors to ensure a secure, controlled, and observable launch on mainnet or other live networks. It guides engineering teams through architecture decisions, role setup, proxy initialization, asset assumptions, monitoring, and emergency procedures, while giving auditors a clear reference to validate that the system is safely upgradeable, correctly configured, and resilient to operational errors or malicious exploits. In short, it is a shared artifact that aligns builders and reviewers on what must be verified before, during, and after deployment to reduce risk, avoid footguns, and support ongoing governance and incident response.

## Table of Contents

[**1. Pre-Deployment (Design & Review)**](#pre-deployment-design--review)
    - [1.1 Architecture & Upgradeability](#11-architecture--upgradeability)
    - [1.2 Threat Model & Deployment Risks](#12-threat-model--deployment-risks)
    - [1.3 Audits & Testing Readiness](#13-audits--testing-readiness)
    - [1.4 External Asset & Token Behavior Validation](#14-external-asset--token-behavior-validation)
[**2. Deployment Plan (Scripts + Human Steps)**](#deployment-plan-scripts--human-steps)
    - [2.1 Deployment script sanity](#21-deployment-script-sanity)
    - [2.2 Proxy Deployment & Initialization Controls](#22-proxy-deployment--initialization-controls)
    - [2.3 Explorer Verification & Alias Consistency](#23-explorer-verification--alias-consistency)
    - [2.4 Admin & Key Management Requirements](#24-admin--key-management-requirements)
[**3. Post-deployment checks (right after mainnet deploy)**](#post-deployment-checks-right-after-mainnet-deploy)
    - [3.1 On-Chain Read-Only Sanity Validation](#31-on-chain-read-only-sanity-validation)
    - [3.2 Functional Smoke Tests](#32-functional-smoke-tests)
    - [3.3 Monitoring, Alerting & Observability Setup](#33-monitoring-alerting--observability-setup)
    - [3.4 Emergency Procedures & Incident Response](#34-emergency-procedures--incident-response)

## 1. Pre-deployment (design & review)

### 1\.1 Architecture & upgradeability

- Are we using proxies? Which pattern? (EIP-1967, UUPS, Transparent, etc.)
- Is there a **single clear admin** (multisig/DAO) for each proxy?
- Is initialization **one-time only** (e.g. OpenZeppelin `initializer` modifiers) and impossible to re-run?
- Have we explicitly listed *all* privileged roles and capabilities (admin, pauser, guardian, minter, oracle manager, etc.)
- If deterministic addresses are required, validate CREATE2/CREATE3 assumptions:
	- confirm salts, address derivation, multi-chain consistency (e.g., Axelar CREATE3 deployer),
	- precompute / verify target contract address before funding or deployment.

### 1\.2 Threat Model & Deployment Risks

- Have we considered MEV / frontrunning during deployment / initialization?
	- Simulate chain-side deployment risks:
		- use Tenderly/Foundry simulations for MEV/frontrun scenarios,
		- explicitly check L2 sequencer assumptions (censorship, delayed inclusion, forced ops).
- Are any “critical” steps done in separate, controlled txs (e.g. set admin, then initialize, etc.)?
- Do we have a plan if deployment tx fails or is partially mined?

### 1\.3 Audits & Testing Readiness

- Contracts audited (including proxy wiring / initialization logic, not just core logic).
- Tests include:
    - Fork tests simulating mainnet state.
    - Tests where init is called unexpectedly / twice / by wrong address.
    - Tests for emergency pause / shutdown.
    - Invariant / fuzz tests for upgrade risk:
		- proxy storage collision testing,
		- metamorphic logic / upgrade paths under Echidna or Foundry invariants.

### 1.4 External Asset & Token Behavior Validation

- For each external token:
    - Have we defined in the spec doc for each network:
        - Token contract address
        - Expected `symbol()`
        - Expected `decimals()`
    - Are we clear on token behavior?
        - For each external asset, confirm that its behavior matches what your protocol expects.
Example: some chains have a “WETH” token address that is not a canonical wrapped ETH contract, it may simply be a standard ERC-20 with no `deposit()` or `withdraw()` function (e.g., certain networks’ WETH tokens behave this way like SAGA).
If your system assumes native wrap/unwrap semantics, this must be verified and documented per network.
	- Cross-reference canonical token metadata (e.g., Chainlink registry / L2Beat asset lists) where available to reduce mismatch risk.

## 2. Deployment plan (scripts + human steps)

### 2.1 Deployment script sanity

- Network IDs, RPC URLs, gas configs correct.
- Deterministic address / salt planning:
	- if using CREATE2/CREATE3, precompute target addresses (`ethers.utils.getCreate2Address`)
	- verify salts and factory deployment ordering.
		- Simulate gas execution paths (`eth_estimateGas` / Hardhat simulation) to avoid mid-deploy OOG failures.
- Script outputs:
    - Implementation address(es)
    - Proxy address(es)
    - Admin / owner addresses
    - Initialization params
- There is NO path where a proxy is deployed with:
    - unset admin, or
    - admin = deployer EOA by accident.

### 2.2 Proxy Deployment & Initialization Controls

- Admin is set first to a secure address (e.g. multisig) before anything else critical.
- Initialization is done via a controlled call:
    - No unnecessary Multicall / batching with third-party contracts.
    - No public `initialize()` that anyone can call if proxy is uninitialized.
- Confirm from logs / explorer:
    - Proxy admin = expected admin.
    - Implementation = expected address.
    - `initialized` flag (if present) is set and cannot be toggled back.
- For multi-chain deployments:
	- ensure coordinated sequencing across chains (manual or relayer-based),
	- verify that ordering is preserved per chain.

### 2.3 Explorer Verification & Alias Consistency

- Implementation contracts verified on Etherscan/other explorers.
- Proxy correctly labeled as proxy + implementation set.
- Cross-check: explorer points to same implementation that your deployment script reports.
- We verify bytecode matches audited commit hash, not just “source compiles.”
- Verify across multi-chain explorers and cross-check with API-derived creation data (e.g., Arbiscan / OptimismScan / `getcontractcreation`).

### 2.4 Admin & Key Management Requirements

- Admin is multisig or DAO, not a single EOA.
- Signers validated and live.
- Secure backups / hardware wallets configured.
- Role assignment (RBAC) executed and recorded.
- Document multisig quorum and rotation cadence; test role transfers in staging.

## 3. Post-deployment checks (right after mainnet deploy)

### 3.1 On-Chain Read-Only Sanity Validation

- For each proxy:
    - `admin()` / `getAdmin()` returns correct admin.
    - `implementation()` returns expected implementation.
    - `initialized` / `version` variables look correct.
- No leftover functions that can:
    - change implementation without admin.
    - re-initialize logic.
- Cross-query via multiple RPC providers to avoid RPC-specific inconsistencies,
- directly verify EIP-1967 slots.

### 3.2 Functional smoke tests

- Tiny “canary” value deposited / traded / minted and then withdrawn.
- Storage layout matches expectations (e.g., view functions returning sane balances, config values).
- Include cross-contract and multi-chain edge interactions (token approval paths, bridge flows if applicable).

### 3.3 Monitoring, Alerting & Observability Setup

- On-chain alerts set for:
    - any implementation upgrade.
    - any admin / role change.
    - abnormal minting / borrowing / swapping thresholds.
- Logs / dashboards ready for launch (e.g. Dune / Flipside / custom).
- Make sure subgraph / indexer ingestion is working (The Graph, custom indexer).
- If on L2, include monitoring of challenge windows, sequencer downtime events.

### 3.4 Emergency Procedures & Incident Response

- Can we pause quickly? Who has pauser role?
- Is everyone aware of the “oh shit” runbook:
    - which contracts to pause,
    - what to tell users (“revoke approvals”, etc.),
    - how to coordinate with CEXs / law enforcement if needed.
- If multi-chain, include cross-domain response procedures (pause bridges / relayers).
- Test dry-run emergency drills quarterly on devnets.
