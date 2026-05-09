# HodlTree

**Role**: Senior Solidity Developer  
**Platforms**: Ethereum · Polygon  
**Period**: 09/2020 – 05/2021

## Overview

Suite of production-grade DeFi protocols spanning flash loans, elastic liquidity management, lending/borrowing, and decentralized exchange infrastructure. Built a diverse ecosystem of smart contracts addressing key DeFi primitives: uncollateralized borrowing via flash loans, multi-token liquidity aggregation with yield strategies, cross-collateral lending with Aave integration, and atomic arbitrage execution across fragmented DEX liquidity. Each protocol deployed to mainnet (Ethereum and/or Polygon) with Certik and internal security reviews.

## Projects

| Project | Stack | Audit |
|---------|-------|-------|
| [Flash Loans 2.0](./flash-loans/) | Solidity 0.7.2 · OpenZeppelin Upgradeable · Truffle | CertiK (June 2021) | 
| [Elastic Modules](./elastic-modules/) | Solidity 0.8.0 · Balancer V2 · Truffle | - |
| [Lend & Borrow](./lend-borrow/) | Solidity 0.8.0 · Aave V2 · Chainlink · Truffle | - |
| [Arbitrage Kit](./arbitrage-kit/) | Solidity 0.8.0 · Multi-DEX Connectors · Truffle | - |
| [HodlSwap](./hodl-swap/) | Solidity 0.8.0 · AMM · EIP-2612 · Truffle | - |

## Technical Highlights

**Flash Loans 2.0** - Multi-token liquidity pool aggregating five stablecoins (sUSD, GUSD, USDC, DAI, TUSD) with 1e36 fixed-point normalized arithmetic. Uncollateralized borrowing via callback pattern with principal + fee repayment in same transaction. Deployed on Ethereum mainnet after CertiK audit (0 Critical/Major findings).

**Elastic Modules** - Dual-pool system (Elastic and Volatile) with automated loss/profit compensation and dynamic fee tiers. Integrates Balancer V2 for multi-asset liquidity management. Supports multiple collateral pairs (sUSD/sETH 75/25, USDC/BAL 50/50) with period-based accounting for fair reward distribution.

**Lend & Borrow** - Dual-contract architecture separating lender and borrower concerns. Lenders deposit wMATIC/wETH to earn Aave yield + borrower fees. Borrowers deposit USDC/DAI collateral and access leverage with dynamic health factor tracking (80% liquidation coefficient, 20% liquidator bonus). Deployed on Ethereum and Polygon with support for multiple collateral denominations.

**Arbitrage Kit** - Atomic arbitrage execution engine across six major DEXs (Uniswap V2/V3, Balancer V1/V2, Curve V1/V2, Bancor, 0x). Delegatecall-based proxy pattern with ArbMemory for stateful cross-call execution. Flash loan callback support for 0-capital arbitrage strategies. Protocol-specific optimizations per DEX (Uniswap V3 tick callbacks, Curve dynamics, Bancor relay networks).

**HodlSwap** - Uniswap V2 fork introducing per-pair configurable swap fees and a flexible protocol fee model. Supports multiple fee-tier pools for the same token pair. Gas‑optimised: TWAP oracle removed, reserves and protocol fee packed into a single storage slot, and multi‑hop routing uses a compact bytes‑encoded path with inline fee decoding. Deterministic pair addresses include the fee in the CREATE2 salt. Demonstrates core AMM enhancements with a strong focus on gas efficiency.