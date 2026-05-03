# BigFan NFT Collectibles Platform

**Company**: Merehead  
**Role**: Senior Solidity Developer  
**Stack**: Solidity · OpenZeppelin Upgradeable · UUPS Proxy · Truffle  
**Period**: 06/2021 - 12/2021

## Overview

Designed and implemented an upgradeable ERC-1155 smart contract powering a fan-oriented NFT marketplace. The contract enables collection management, card minting, peer-to-peer trading, and configurable royalty fees - all with an innovative on-chain IPFS content addressing system that eliminates per-token metadata storage costs.

## Technical Scope

### Collection & Card Management
- ERC-1155 multi-token standard enabling flexible token types within a single contract
- Admin-controlled collection creation with supply caps and base prices
- Batch minting and transfer operations for gas efficiency
- Individual card-level metadata and pricing

### Marketplace Functionality
- ETH-native trading: direct purchase from contract owner, peer-to-peer swaps via atomic `swapEthToToken`
- Two-tier royalty fee system (collection-level and card-level, with card taking precedence)
- Fee routing to configurable recipient addresses on each sale
- OpenSea compatibility: contractURI for collection metadata, proxy account whitelisting for gasless listings

### On-Chain IPFS Metadata
- **CIDv0 encoder (`CIDLib.sol`)**: Bidirectional IPFS hash ↔ uint256 mapping in pure Solidity
- Token IDs encode complete IPFS content hashes (multihash format with `0x1220` prefix for SHA2-256)
- `tokenURI()` computes the full CIDv0 string on-chain from token ID, zero storage overhead
- Deterministic and gas-bounded, no external calls required

### Role-Based Access Control
- Three roles via OpenZeppelin `AccessControlEnumerable`: DEFAULT_ADMIN, MINTER, PAUSER
- Emergency pause functionality halts all transfers atomically
- Clear separation of concerns: minting, trading, fee management

## Key Engineering Decisions

| Decision | Rationale |
|----------|-----------|
| ERC-1155 instead of ERC-721 | Enables efficient batch operations and multiple token types in single contract |
| On-chain CIDv0 encoding | Token ID itself stores content hash; eliminates per-token URI storage and guarantees content integrity |
| UUPS proxy with upgradeable base | Composable upgradeable modules (ERC1155, AccessControl, Ownable, Pausable) with `__gap[50]` for safe future extensions |
| Two-tier fee resolution | Card-specific fees take precedence; collection fees as fallback; zero as default - resolved at purchase time |
| Polygon as primary target | Low gas fees for high-volume fan engagement and trading scenarios |

## Technical Insights

The **on-chain CIDv0 encoder** is the core innovation: instead of storing IPFS hashes as strings, token IDs _are_ the multihash. The encoder prepends the `0x1220` multihash prefix (SHA2-256, 32 bytes) and runs a full Base58 encoding pipeline in pure Solidity - no external calls, fully deterministic, and gas-bounded for all 32-byte inputs.

The **upgradeable architecture** inherits from a custom base class that composes five OpenZeppelin upgradeable modules through the `initializer` pattern. Storage layout extensibility is guaranteed with a reserved `__gap[50]` array, enabling safe future storage additions without layout shifts.

Fee resolution implements a **fallback hierarchy**: check card-specific fee → collection-level fee → zero, all resolved at purchase time without off-chain coordination.

## Links & Archives

- **GitHub**: Not public (company policy)
- **Contracts Deployed**: Prototype — not deployed to mainnet

See [Merehead page](../README.md) for additional context.
