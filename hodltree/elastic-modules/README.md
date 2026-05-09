# Elastic Modules

**Company**: HodlTree  
**Role**: Senior Solidity Developer  
**Stack**: Solidity 0.8.0 · Truffle · Balancer V2 · OpenZeppelin Contracts · Web3.js  
**Period**: 09/2020 - 05/2021

## Overview

Sophisticated Ethereum DeFi protocol implementing elastic and volatile liquidity pools with automated loss/profit compensation mechanisms. The system integrates with Balancer Protocol to enable dynamic yield strategies with configurable risk parameters and deterministic contract deployment. Two primary trading pairs deployed: sUSD/sETH (75/25 ratio) and USDC/BAL (50/50 ratio), each with independent liquidity management and collateral tracking.

## Technical Scope

### Elastic Pool (EPT Token)

- ERC20 token representing pool shares with automatic loss/profit compensation
- Configurable compensation limit (default 50%) protecting against unrealized losses
- Excess profits split between platform fees and worker incentives
- Time-based withdrawal holds for capital stability
- Real-time balance reconciliation with Balancer pools

### Volatile Pool (VPT Token)

- ERC20 token minted based on deposit coefficient independent of time
- Tiered yield strategy capturing volatility premiums
- Seamless composition with Elastic Pool for multi-strategy access
- Dynamic rebalancing through multi-pool swap chains

### Balancer Protocol Integration

- Joins and exits Balancer pools (USDC-WETH for sUSD/sETH; BAL routing for USDC/BAL)
- BAL token reward distribution with automated swaps via intermediate routing
- Dynamic rebalancing through multi-pool swap chains
- Real-time pool data updates for fair pricing

### Deterministic Contract Deployment via Factory

- All four contracts have circular constructor-argument dependencies - each requires the addresses of contracts not yet deployed
- Factory acts as sole deployer, pre-computing all addresses via CREATE deterministic formula (`keccak256(rlp([deployer, nonce]))`) before any contract exists
- All four contracts deployed atomically in a single transaction with cross-references already resolved
- Each deployed address verified post-construction against expected value

### Risk Management & Safety

- Reentrancy guards on all state-changing functions
- Safe token transfers with explicit error handling
- Emergency stop mechanism with cascading control
- Owner-based access control with multi-signature capability

## Key Engineering Decisions

| Decision | Rationale |
|----------|-----------|
| Balancer V2 over Uniswap | Multi-asset pool support enables more flexible collateral ratios (75/25, 50/50) without AMM constant product constraints |
| Deterministic deployment via CREATE (Factory) | Sequential deployment is structurally impossible due to circular constructor-argument dependencies; Factory pre-computes all addresses before any contract is live |
| Three-tier fee architecture | Aligns incentives between platform, workers, and users; loss compensation funded from performance profits |
| Time-based withdrawal holds | Prevents flash loan attacks and encourages long-term capital commitment |
| Period-based accounting (VPStorage) | Enables fair distribution of BAL rewards and platform fees across different entry times without per-user iteration |

## Links & Archives

- **GitHub**: Not public (company policy)
- **Contracts Deployed**:
  - **sUSD/sETH 75/25 v1.0**: <a href="https://etherscan.io/address/0x95142849d31eaa20b5b9ab746dff27ff400ce6bf#code" target="_blank" rel="noopener noreferrer">ElasticPool</a> · <a href="https://etherscan.io/address/0x3201a8e9874bbf7ba9d6e1c3339961abc8f8b29e#code" target="_blank" rel="noopener noreferrer">VolatilePool</a> · <a href="https://etherscan.io/address/0xce596bf99d21e46fa91143c03d7a356682b67859#code" target="_blank" rel="noopener noreferrer">ReservePool</a> · <a href="https://etherscan.io/address/0xb7ead8c418f3d03bc22dd538c22600abe7209e72#code" target="_blank" rel="noopener noreferrer">VPStorage</a>
  - **USDC/BAL 50/50 v1.0**: <a href="https://etherscan.io/address/0x78E52d69fA8e0F036fFEF0BcDc4C289DB0DF63E2#code" target="_blank" rel="noopener noreferrer">ElasticPool</a> · <a href="https://etherscan.io/address/0x55Dbe7Df69e9385b9bE0f4BdCc9f1505CD139791#code" target="_blank" rel="noopener noreferrer">VolatilePool</a> · <a href="https://etherscan.io/address/0x87B46E49681E08E3adDF8A90F6a1fb5183079033#code" target="_blank" rel="noopener noreferrer">ReservePool</a> · <a href="https://etherscan.io/address/0xcB72e764Ab46535aAD13cbF55b1F06cB15347A95#code" target="_blank" rel="noopener noreferrer">VPStorage</a>
- **Articles**: <a href="https://medium.com/hodltree/hodltree-elastic-modules-explained-24b86b3ec192" target="_blank" rel="noopener noreferrer">Medium - HodlTree: Elastic Modules Explained</a> - [Local Copy](./archived/hodltree-medium-elastic-modules.html)

## Technical Insights

  - EPT is a loss-compensation voucher, not an LP token. Elastic Pool positions are tracked internally per-user - no ERC20 tokens are minted
  on deposit. EPT tokens are minted only on withdrawal at a loss, functioning as a typed claim against the reserve. Users redeem them at
  ReservePool 1:1 for the underlying by burning. This cleanly separates LP accounting from the insurance claim layer.

  - Volatile Pool depositors are the protocol's implicit risk underwriters. Their USDC flows directly to ReservePool, funding EPT redemptions.
   The 1.5× VPT mint coefficient makes the asymmetry explicit: Volatile Pool depositors receive amplified exposure to BAL rewards in
  exchange for bearing the counterparty risk of ElasticPool's downside protection.

  See [HodlTree page](../README.md) for additional context.