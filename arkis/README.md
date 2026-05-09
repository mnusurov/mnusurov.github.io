# DeFi Prime Broker

**Company**: Arkis
**Role**: Senior Solidity Developer (sole smart contract engineer)
**Stack**: Solidity 0.8.22 · Diamond (ERC-2535) · Foundry · Hardhat · TypeScript · Tenderly
**Period**: 06/2023 - 10/2025

## Overview

Under-collateralized leveraged trading protocol on Ethereum designed for institutional use. Borrowers open multi-protocol margin positions across Uniswap, Aave, Morpho, CurveFi, ConvexFi, Pendle, and 1inch from a single risk-controlled account; lenders deploy capital into Agreement contracts and earn yield on leveraged trading flow. Joined as sole smart contract engineer, drove all subsequent development, prepared the codebase for four independent audits (Quantstamp, Trail of Bits, Cantina, Spearbit), authored the DefiLlama TVL adapter, and built three Dune dashboards for on-chain state monitoring and audit preparation.

## Technical Scope

### Margin Account System
- Isolated per-user accounts deployed via beacon proxy with an object-pool factory: closed accounts return to a per-owner pool instead of being destroyed, cutting redeployment cost for repeat users.
- Formal state machine (`UNDEFINED -> REGISTERED -> OPENED -> SUSPENDED -> CLOSED`) with bitmap-based state composition for multi-state checks and same-block reentrancy guards.
- Reusable `StateMachine.sol` primitive (power-of-2 state IDs, transition map) shared across Agreements, Accounts, and Vaults.

### JIT Compiler & Multi-Protocol Integration
- Just-in-time compiler that transforms high-level trading scripts into validated EVM command sequences, with protocol-specific evaluators for Uniswap, Aave, Morpho, CurveFi, ConvexFi, Pendle, and 1inch (v5 + v6).
- Dynamic amount resolution: `ExchangeAll`-style instructions resolve balance amounts at execution time inside the account context.
- Compliance enforced at compile time: scripts referencing non-whitelisted tokens, pools, or operators fail before any transaction reaches the chain, avoiding gas waste on predictably-reverting calls.

### Two-Sided Lending (Agreements & Vaults)
- `Agreement` contracts: lenders deposit leverage capital, borrowers rent it to fund margin positions. Per-second APY accrual using fixed-point precision; role-based access cleanly separates lender and borrower permissions.
- `AgreementStaking` and `VaultStaking` implemented from scratch as ERC-4626 tokenized vaults with checkpoint-based accounting. Delivered two full iterations of `AgreementStaking` to match evolving product requirements.

### Compliance & Whitelisting
- `WhitelistingController` enforces per-agreement token, protocol, and operator whitelists. Each protocol has a dedicated validator contract that validates routes, pool parameters, and position sizes before execution.
- Two-layer enforcement: validators run during JIT compilation (fail-fast, pre-transaction) AND again at execution time inside the account, regardless of how instructions were assembled. Bypassing the compiler does not bypass compliance.

### Liquidation Engine
- Two-phase suspension-then-liquidation pipeline with deterministic liquidation plans, non-reverting partial execution, and account state recovery.
- Operator-driven liquidation roles separated from core protocol roles.

### Deployment Infrastructure
- Single Hardhat Deploy codebase handles four scenarios without branching: production upgrade, fresh mainnet deploy, fork of production for integration testing, and end-to-end testing of the deploy process itself inside a fork.
- Tenderly virtual networks integrated into GitHub Actions CI for fork simulations and E2E scenario testing.
- 168 Solidity unit tests (Forge) + E2E suite that spins up Anvil, deploys (or upgrades) via Hardhat Deploy, then runs Forge tests against the live deployment.
- NatSpec across all interfaces; Foundry's doc generator publishes HTML docs via GitHub Actions.

### DefiLlama TVL Adapter
- JavaScript adapter merged into the official DefiLlama-adapters repo. Tracks Arkis TVL across Ethereum and Hyperliquid (native HYPE, WHYPE, stHYPE in wrapped vaults).
- Discovers leverage agreements and margin accounts dynamically via factory event logs (`AgreementCreated`, `AccountDeployed`) from block 21,069,508 onward, eliminating hard-coded contract lists.
- Separates borrowed-asset accounting from collateral, supports unwrapping LP tokens and resolving nested token pairs, uses multiCall batching and incremental log caching to minimize RPC load.

### Dune Analytics Dashboards
- Three SQL dashboards built for audit preparation and on-chain state verification: contract ownership/roles, per-account inspection, and whitelist configuration.
- **Contracts Dashboard**: queries current ownership and role assignments across all protocol contracts and EOAs via a fixed address list and role-hash lookups; discovers margin accounts and agreements via factory deploy events (`AgreementCreated`, `AccountDeployed`), including borrower and lender role mappings per agreement.
- **Margin Account Inspector**: dropdown selector for any deployed margin account - displays status, current and previous owner, linked agreement address, last registration and status-change timestamps, and token balance breakdown.
- **Whitelist State**: resolves current system-wide whitelisted protocols, tokens, and operators from a fixed list of name hashes, providing a point-in-time snapshot of the compliance configuration.

## Key Engineering Decisions

| Decision | Rationale |
|----------|-----------|
| ERC-2535 Diamond proxy for `Dispatcher` and `Compiler` | Bypasses contract bytecode size limits while keeping upgrade logic modular; per-facet selector routing isolates feature surface area. |
| ERC-7201 namespaced storage everywhere | Prevents storage collisions across proxy layers and diamond facets; assembly-based direct slot access. |
| Two-layer compliance (compile-time + execution-time) | Compile-time fail-fast eliminates wasted gas on predictably-invalid operations; execution-time check makes bypass structurally impossible. |
| Account object pooling in `AccountFactory` | Closed accounts return to a per-owner pool instead of being destroyed - significant gas savings for repeat traders. |
| ERC-4626 for `AgreementStaking` and `VaultStaking` | Standardized share accounting and external composability over custom share math. |
| Hybrid Hardhat + Foundry pipeline | Hardhat Deploy handles upgrade and deployment workflows; Foundry runs the unit + E2E test suite. One deploy codebase covers production, fresh mainnet, fork-of-prod, and self-test scenarios. |

## Audit Record

| Auditor | Date | Scope | Link | Archived |
|---------|------|-------|------|----------|
| Quantstamp | Dec 2023 | General security review | <a href="https://certificate.quantstamp.com/full/arkis/07924f14-1727-4e72-8e7a-2bc71aa735dd/index.html" target="_blank" rel="noopener noreferrer">Report</a> | [Local Copy](./archived/Quantstamp%20Arkis%20Report%202023.html) |
| Trail of Bits | Dec 2024 | ERC-2535 Diamond pattern, leverage protocol logic | <a href="https://github.com/trailofbits/publications/blob/08a49dbe75605043934d044ead8f72ff04e37ff1/reviews/2024-12-arkis-defi-prime-brokerage-securityreview.pdf" target="_blank" rel="noopener noreferrer">Report</a> | [Local Copy](./archived/TrailOfBits%20Arkis%20Prime%20Brokerage%20Final%20Report.pdf) |
| Cantina | May 2025 | Smart contract security review | <a href="https://cantina.xyz/portfolio/6adef5e1-9694-4649-8fa6-0f5c2d6bc9eb" target="_blank" rel="noopener noreferrer">Report</a> | [Local Copy](./archived/Cantina%20Arkis%20Smart%20Contracts%20May%202025.pdf) |
| Cantina + Spearbit | Sep 2025 | Joint competitive review, threat modeling, edge cases | <a href="https://docs.arkis.xyz/home/security/december-2025-audit" target="_blank" rel="noopener noreferrer">Report</a> | [Local Copy](./archived/Cantina%20Audits%20Sept%20Dec%202025.pdf) |

## Links & Archives

- **GitHub**: Not public (company policy)
- **Contracts Deployed**: Diamond modules and 60+ verified facets/validators/evaluators on Ethereum mainnet - [full list](./archived/contract-addresses.md)
- **DefiLlama**: <a href="https://defillama.com/protocol/arkis" target="_blank" rel="noopener noreferrer">defillama.com/protocol/arkis</a>
- **Dune**:
  - <a href="https://dune.com/mnusurov/arkis-contracts-dashboard" target="_blank" rel="noopener noreferrer">Contracts Dashboard</a>
  - <a href="https://dune.com/mnusurov/arkis-margin-account" target="_blank" rel="noopener noreferrer">Margin Account</a>
  - <a href="https://dune.com/mnusurov/arkiswhitelist" target="_blank" rel="noopener noreferrer">Whitelist</a>

## Technical Insights

- **Compliance enforced twice on purpose.** Running validators at compile time eliminates predictably-failing transactions entirely, but the execution-time check is what makes the system safe: even if someone hand-assembles EVM commands and skips the compiler, the account itself rejects them. Compile-time is performance, execution-time is the security boundary.
- **Object-pooled accounts pay off for active users.** Beacon-proxy accounts are cheap to deploy, but for traders who open and close positions repeatedly the pool removes deployment cost from the hot path entirely. The pool is per-owner, so liveness for one user does not affect another.
- **One deploy codebase for four scenarios.** Production upgrades, fresh mainnet deploys, integration testing on a prod fork, and E2E testing the deploy process itself all run through the same Hardhat Deploy scripts without per-scenario branches. Prevented the usual drift between "what the deploy script does" and "what we run in CI".
- **Two iterations of AgreementStaking from scratch.** The first ERC-4626 staking implementation matched product spec at the time; six months later product requirements had shifted enough that a clean rewrite was cheaper than retrofitting. Both iterations live in the codebase; migration handled via the standard upgrade path.