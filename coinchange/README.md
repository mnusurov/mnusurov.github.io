# Atomic Execution Framework

**Company**: Coinchange  
**Role**: Senior Solidity Developer  
**Stack**: Solidity 0.8.9 · Hardhat · OpenZeppelin · Typechain  
**Period**: 12/2021 - 06/2023

## Overview

Built an atomic execution framework enabling multi-DeFi operations in a single transaction across Ethereum, BSC, Polygon, and Avalanche. The framework reduced transaction costs through optimized calldata encoding and deterministic account deployment via CREATE2/CREATE3 mechanisms. Mentored 3 Solidity developers on testing patterns, security, and gas optimization.

## Technical Scope

### Atomic Execution Framework
- Designed custom ABI encoding reducing calldata overhead by ~20%
- Implemented multi-operation orchestration (Uniswap, Curve, Balancer, Aave, Yearn)
- Enabled atomic execution, eliminating inter-operation execution risk within a single transaction

### Account Deployment Mechanism
- Deployed EIP-1167 minimal proxies for personal accounts
- Implemented CREATE2 + user address salt for deterministic, vanity-address-capable account creation
- Engineered gasless self-deployment flows without upfront capital requirements

### Infrastructure & Migration
- Architected CREATE3Factory for vanity address deployment independent of nonce/bytecode
- Led Truffle to Hardhat migration including all tests and deployment scripts

## Key Engineering Decisions

| Decision | Rationale |
|----------|-----------|
| Salt-based CREATE2 determinism | User address as salt enabled predictable account addresses, simplified lookup, and gasless onboarding |
| Custom calldata encoding | Reduced transaction costs per operation by ~20% through tailored ABI encoding |
| Minimal proxy pattern (EIP-1167) | Minimized bytecode overhead for account factory at scale |
| Hardhat migration | Better test performance and deployment flexibility vs. Truffle |

## Links & Archives

- **GitHub**: Not public (company policy)

## Technical Insights

Counterfactual deployment made retail onboarding practical: because the account address is derived deterministically from the user's address via CREATE2, the account exists as a known address before it is ever deployed. Users fund that address first; deployment happens later - triggered by the user or a relayer - with no ETH required in the deployer's wallet at signup. This pattern eliminated the "chicken-and-egg" problem of needing gas to get an account before having funds to pay for gas.
