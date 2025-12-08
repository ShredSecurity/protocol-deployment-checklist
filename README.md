# Protocol Deployment Checklist

## A. Pre-deployment (design & review)

### 1\. Architecture & upgradeability

- Are we using proxies? Which pattern? (EIP-1967, UUPS, Transparent, etc.)
- Is there a **single clear admin** (multisig/DAO) for each proxy?
- Is initialization **one-time only** (e.g. OpenZeppelin `initializer` modifiers) and impossible to re-run?
- Have we explicitly listed *all* privileged roles and capabilities (admin, pauser, guardian, minter, oracle manager, etc.)

### 2\. Threat model for deployment

- Have we considered MEV / frontrunning during deployment / initialization?
- Are any “critical” steps done in separate, controlled txs (e.g. set admin, then initialize, etc.)?
- Do we have a plan if deployment tx fails or is partially mined?

### 3\. Audits & tests

- Contracts audited (including proxy wiring / initialization logic, not just core logic).
- Tests include:
    - Fork tests simulating mainnet state.
    - Tests where init is called unexpectedly / twice / by wrong address.
    - Tests for emergency pause / shutdown.


## B. Deployment plan (scripts + human steps)

### 1\. Deployment script sanity

- Network IDs, RPC URLs, gas configs correct.
- Nonce plan (esp. if protocol will be deployed on multiple chains and the addresses on all chains should be the same).
- Script outputs:
    - Implementation address(es)
    - Proxy address(es)
    - Admin / owner addresses
    - Initialization params
- There is NO path where a proxy is deployed with:
    - unset admin, or
    - admin = deployer EOA by accident.

### 2\. Proxy deployment & initialization

- Admin is set first to a secure address (e.g. multisig) before anything else critical.
- Initialization is done via a controlled call:
    - No unnecessary Multicall / batching with third-party contracts.
    - No public `initialize()` that anyone can call if proxy is uninitialized.
- Confirm from logs / explorer:
    - Proxy admin = expected admin.
    - Implementation = expected address.
    - `initialized` flag (if present) is set and cannot be toggled back.

### 3\. Explorer verification & aliasing

- Implementation contracts verified on Etherscan/other explorers.
- Proxy correctly labeled as proxy + implementation set.
- Cross-check: explorer points to same implementation that your deployment script reports.
- We verify bytecode matches audited commit hash, not just “source compiles.”

### 4\. Admin & key management

- Admin is multisig or DAO, not a single EOA.
- Signers validated and live.
- Secure backups / hardware wallets configured.
- Role assignment (RBAC) executed and recorded.


## C. Post-deployment checks (right after mainnet deploy)

### 1\. Read-only sanity checks

- For each proxy:
    - `admin()` / `getAdmin()` returns correct admin.
    - `implementation()` returns expected implementation.
    - `initialized` / `version` variables look correct.
- No leftover functions that can:
    - change implementation without admin.
    - re-initialize logic.

### 2\. Functional smoke tests

- Tiny “canary” value deposited / traded / minted and then withdrawn.
- Storage layout matches expectations (e.g., view functions returning sane balances, config values).

### 3\. Monitoring & alerting

- On-chain alerts set for:
    - any implementation upgrade.
    - any admin / role change.
    - abnormal minting / borrowing / swapping thresholds.
- Logs / dashboards ready for launch (e.g. Dune / Flipside / custom).

### 4\. Emergency procedures

- Can we pause quickly? Who has pauser role?
- Is everyone aware of the “oh shit” runbook:
    - which contracts to pause,
    - what to tell users (“revoke approvals”, etc.),
    - how to coordinate with CEXs / law enforcement if needed.
