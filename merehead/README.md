# Merehead

**Role**: Senior Solidity Developer  
**Platforms**: Ethereum · Polygon · Aurora  
**Period**: 06/2021 - 12/2021

## Overview

Designed and implemented smart contracts for two flagship NFT and DAO projects: BigFan (a fan-oriented NFT marketplace platform) and reBaked (a decentralized project management and payment protocol). Work spanned full contract architecture, on-chain metadata systems, and payment automation.

## Projects

| Project | Stack | Focus |
|---------|-------|-------|
| [BigFan NFT](./bigFan/) | Solidity · OpenZeppelin · UUPS Proxy | ERC-1155, marketplace, on-chain IPFS encoding |
| [reBaked DAO](./reBaked/) | Solidity · ERC-20 · AccessControl | Decentralized project management, payment distribution |

## Links & Archives

- **GitHub**: Not public (company policy)
- **Contracts Deployed**: Not public (company policy)

## Technical Insights

**reBaked DAO** – Decentralised project management and payment protocol for milestone-driven collaboration between initiators, contributors and observers. On-chain enforcement of a dual payment model: Minimum Guaranteed Payments (MGP) plus performance bonuses distributed proportionally via PPM scoring. Library-based architecture with collision-resistant ID generation, triple-nested mapping for efficient state, and batch operations for gas savings. Supports any ERC‑20 token, optional on‑demand token factory, and reentrancy‑safe payment claims.

**BigFan NFT** – ERC-1155 marketplace for fan-oriented digital collectibles. Core innovation: on-chain CIDv0 encoder encodes complete IPFS content hashes directly into token IDs, eliminating per‑token metadata storage while guaranteeing content integrity. Supports collection/card management, ETH‑native trading with atomic swaps, and a two‑tier royalty system. Built with UUPS proxy for composable upgradeable modules and OpenSea compatibility.

