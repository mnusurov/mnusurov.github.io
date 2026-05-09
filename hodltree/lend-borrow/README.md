# Lend & Borrow Protocol

**Company**: HodlTree  
**Role**: Senior Solidity Developer  
**Stack**: Solidity 0.8.0 · Aave V2 · Chainlink Oracle · Truffle · OpenZeppelin · Web3.js  
**Period**: 09/2020 - 05/2021

## Overview

Decentralized lending and borrowing protocol on Polygon and Ethereum enabling users to deposit collateral, earn multi-layer yields through Aave integration, and borrow against their holdings with dynamic risk management. The system uses Chainlink price oracles for accurate asset pricing and implements sophisticated liquidation mechanics to maintain protocol health. Supports multiple collateral types (USDC, DAI) and denominations (wMATIC, wETH) with per-token yield strategies.

## Technical Scope

### Dual-Contract Architecture

- **Lender Contract** - Manages deposits and yield accrual: users deposit wrapped assets (wMATIC, wETH) to mint pool tokens proportional to share; earn yield from Aave deposit interest and borrower fees; proportional redemption with no lockup period; ERC20-compliant pool token representing lender share
- **Borrower Contract** - Manages collateral and debt: users deposit supported collateral (USDC, DAI) to establish borrowing capacity; borrow wMATIC or wETH up to configurable collateral coefficient (default 60%); real-time health factor tracking with liquidation at factor below 1.0; repay debt to withdraw collateral proportionally

### Aave Yield Integration

- Deposited collateral automatically lent to Aave V2 for additional yield
- Aave rewards claimed and distributed to protocol owner
- Multi-layer yield: Aave interest + borrower fees accumulated in lender pool
- Supports Aave incentive programs with automated reward claiming

### Dynamic Risk Management

- Real-time collateral valuation via Chainlink oracle (8-decimal feeds converted to 18-decimal precision)
- Health factor calculation: (collateral value × collateral coefficient) / debt
- Position becomes liquidatable when health factor drops below 1.0
- Liquidation threshold at 80% collateral coefficient with 20% liquidator bonus incentive

### Liquidation Mechanism

- Any user can liquidate undercollateralized positions
- Liquidator repays debt and receives discounted collateral (20% bonus)
- Automatic health factor verification preventing bad liquidations
- Incentive structure maintains protocol solvency while rewarding liquidators

### Fee Distribution System

- Off-chain fee calculation via ECDSA-signed messages
- On-chain verification maintains security without expensive computation
- Lenders claim accumulated fees using signed messages
- Admin-controlled fee rate with transparent accrual tracking

## Key Engineering Decisions

| Decision | Rationale |
|----------|-----------|
| Dual-contract (Lender/Borrower) | Clear separation of concerns; lenders unaware of borrower risk; independent token economics |
| Aave collateral deposits | Free yield generation on locked collateral; leverages battle-tested Aave V2 risk management |
| Chainlink oracle feeds | Decentralized price feeds eliminate single-point-of-failure pricing; 8-decimal standardization reduces implementation risk |
| ECDSA fee signatures | Off-chain computation reduces gas costs while maintaining on-chain verification; enables batched fee claims |
| 20% liquidation bonus | Aggressive incentive attracts liquidators; maintains protocol solvency during market volatility |

## Links & Archives

- **GitHub**: Not public (company policy)
- **Contracts Deployed**:
  - **Ethereum wETH**: <a href="https://etherscan.io/address/0xb3e1912fa5d9d219da8c65cda407cc998849428b#code" target="_blank" rel="noopener noreferrer">Lender</a> · <a href="https://etherscan.io/address/0x8ac9425260b6da02db07da7980b09525ebf3b6a0#code" target="_blank" rel="noopener noreferrer">Borrower USDC</a> · <a href="https://etherscan.io/address/0x45d5a790da3bfa305efca81eac652678ae3a90a6#code" target="_blank" rel="noopener noreferrer">Borrower DAI</a>
  - **Polygon wMATIC**: <a href="https://polygonscan.com/address/0xed3DC4478c3043Ad4FF44e293271a679CD4a3c9D#code" target="_blank" rel="noopener noreferrer">Lender</a> · <a href="https://polygonscan.com/address/0xF6d455c656CF60c88564395517bE5652388Ae743#code" target="_blank" rel="noopener noreferrer">Borrower USDC</a> · <a href="https://polygonscan.com/address/0x69903eD03FE69F6ee704163476c36A16CdDE958a#code" target="_blank" rel="noopener noreferrer">Borrower DAI</a>
  - **Polygon wETH**: <a href="https://polygonscan.com/address/0x1af02Ff7FFf08a8f459F5f8eC198E2a6BB8a928D#code" target="_blank" rel="noopener noreferrer">Lender</a> · <a href="https://polygonscan.com/address/0x7235c717e5291e076CFCcB07FB0000466233dEAF#code" target="_blank" rel="noopener noreferrer">Borrower USDC</a> · <a href="https://polygonscan.com/address/0x2791bca1f2de4661ed88a30c99a7a9449aa84174#code" target="_blank" rel="noopener noreferrer">Borrower DAI</a>
- **Articles**: <a href="https://medium.com/hodltree/20-for-lenders-0-for-borrowers-is-it-possible-absolutely-57c396a1fb18" target="_blank" rel="noopener noreferrer">Medium - 20% for Lenders, 0% for Borrowers?</a> - [Local Copy](./archived/hodltree-medium-lend-borrow.html)

## Technical Insights

- **Chainlink decimal inversion in one step.** Price feeds return USD values in 8-decimal precision; collateral is 6-decimal USDC, debt is 18-decimal wMATIC. Rather than converting in stages, `priceDecimalsPower = 18 + oracle_decimals` is computed once at deploy time, and `price = 10 ** priceDecimalsPower / rawFeedValue` normalizes and inverts in a single expression - no intermediate rounding.

- **Yield accounting via aToken delta.** User collateral is tracked as `totalBalance`. The same funds sit in Aave as a rebasing `aToken`, whose balance grows automatically with interest. The difference `aToken.balanceOf(this) − totalBalance` is the claimable yield - no separate accrual logic needed.

- **Mainnet-fork-first integration tests.** The test suite forks Polygon mainnet with real Aave and Chainlink addresses and an unlocked USDC whale account. All external integrations are tested against production state, not mocks - a decision that caught decimal-handling bugs invisible in unit tests.

- **Atomic compound operations.** `depositAndBorrow()` and `repayAndWithdraw()` execute both actions in one transaction, eliminating the window between two separate calls where a price move could leave the position undercollateralized before the second call lands.

See [HodlTree page](../README.md) for additional context.