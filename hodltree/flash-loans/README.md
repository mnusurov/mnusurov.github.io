# Flash Loans 2.0

**Company**: HodlTree  
**Role**: Senior Solidity Developer  
**Stack**: Solidity 0.7.2 · OpenZeppelin Upgradeable · Truffle · Mocha/Chai · Web3.js  
**Period**: 09/2020 - 06/2021

## Overview

Production-grade Ethereum smart contract protocol enabling flash loans and cross-stablecoin swaps through a sophisticated multi-token liquidity pool. The system aggregates five major stablecoins (sUSD, GUSD, USDC, DAI, TUSD) with decimal normalization to manage different token standards. Deployed on mainnet after Certik audit, supporting real-world stablecoin liquidity aggregation and flash loan arbitrage strategies with exhaustive test coverage including mainnet fork testing.

## Technical Scope

### Multi-Token Liquidity Pool

- Aggregates five ERC20 stablecoins with automatic decimal normalization via configurable multipliers and 1e36 fixed-point precision
- Flexible withdrawal modes: proportional (preserve pool balance), by percentage, or exact output specification
- Virtual price calculation for LP token (hFLP-USD) valuation - real-time market pricing for fair slippage estimation
- OpenZeppelin upgradeable proxy pattern (V1 → V2 → V3) enabling live protocol improvements without migration

### Flash Loan Mechanism

- Uncollateralized borrow via callback pattern; principal + fee repaid within same transaction
- Fee splitting between protocol and admin with cumulative multi-token balance tracking
- Reentrancy guards + pausable contract + role-based access control (admin/pauser roles)

### Exchange Contract

- Atomic cross-stablecoin swaps using flash loans as liquidity source with minimal slippage
- Low-level calldata encoding for callback integration; input tokens recovered without pre-approval requirement

## Key Engineering Decisions

| Decision | Rationale |
|----------|-----------|
| OpenZeppelin upgradeable proxies | Enable live protocol improvements after mainnet deployment without state migration |
| 1e36 fixed-point precision | Handles arbitrary token decimals (6-18) across five stablecoins without precision loss |
| Role-based access control | Separate admin and pauser roles limit blast radius of key compromise |
| Exchange via flash loan callback | Atomic swap with zero capital requirement; single transaction execution avoids MEV exposure |

## Audit Record

| Auditor | Date | Scope | Link | Archived |
|---------|------|-------|------|----------|
| CertiK | June 2021 | Flash loan mechanism, reentrancy guards, access control | <a href="https://skynet.certik.com/projects/hodltree" target="_blank" rel="noopener noreferrer">Skynet</a> | [Local Copy](./archived/certik-audit-jun2021.pdf) |

## Links & Archives

- **GitHub**: <a href="https://github.com/HodlTreeProtocol/stableFlashloan" target="_blank" rel="noopener noreferrer">Repository</a> - [Local Copy](./archived/stableFlashloan-repo/)
- **Contracts Deployed**:
  - <a href="https://etherscan.io/address/0x2e5a08c26cb22109e585784c4f99363bb3e199ab#code" target="_blank" rel="noopener noreferrer">LiquidityPoolProxy (Ethereum)</a>
  - <a href="https://etherscan.io/address/0x7821eade727c8c6eb58977b037e69c44675591c6#code" target="_blank" rel="noopener noreferrer">LiquidityPool (Ethereum)</a>
  - <a href="https://etherscan.io/address/0x87bb770cdc7B1E71EaD5C2904a2081C4a80f252b#code" target="_blank" rel="noopener noreferrer">Exchange (Ethereum)</a>
  - <a href="https://polygonscan.com/address/0xCAFDa65B1031535F1766C6b1E3b5efF5520c7C0f#code" target="_blank" rel="noopener noreferrer">LiquidityPoolProxy (Polygon)</a>
  - <a href="https://polygonscan.com/address/0xdf418333e269041bd222ef7f798cfa4cd4debf35#code" target="_blank" rel="noopener noreferrer">LiquidityPool (Polygon)</a>
  - <a href="https://polygonscan.com/address/0x0127afD09CfD814938A9c292970DbC2EBD86ddf4#code" target="_blank" rel="noopener noreferrer">Exchange (Polygon)</a>
- **Articles**: <a href="https://medium.com/hodltree/hodltree-introducing-flash-loans-2-0-b3829b2e5424" target="_blank" rel="noopener noreferrer">Medium - HodlTree: Introducing Flash Loans 2.0</a> - [Local Copy](./archived/hodltree-medium-flash-loans-2.0.html)

## Technical Insights

- **Cross-token fee accounting without rounding loss:** Pool fees accrue in five stablecoins with different decimals, requiring a cumulative admin balance tracker with 1e36 precision and proportional withdrawal logic that fairly distributes fees to any recipient address without dust accumulation.
- **Callback isolation despite multiple external calls:** Flash loan borrow flow involves multiple external token transfers; the reentrancy guard was applied around the entire borrow–callback–repay sequence, and the callback target’s execution is logically isolated so that state corruption is prevented even if a malicious callback attempts re-entry.

See [HodlTree page](../README.md) for additional context.