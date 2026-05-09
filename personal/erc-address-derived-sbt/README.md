# ERC-XXXX: Address-Derived Non-Transferable Token

**Company**: Personal project
**Role**: Author
**Stack**: Solidity 0.8.35 · Foundry · ERC-165 · ERC-721 · ERC-5192
**Period**: 01/2026 – present

## Overview

Draft ERC standard for soulbound tokens where tokenId derives deterministically
from owner address XOR contract address: `tokenId = uint256(uint160(owner)) ^ uint160(address(this))`.
Enforces one-token-per-address at protocol level with minimal storage footprint.

## Technical Scope

### Deterministic Token ID
- `tokenId = owner XOR contract address` - unique per owner per contract, no sequential counter
- Single `mapping(uint256 => bool)` for existence tracking instead of owner mapping

### Non-Transferability
- Transfer, approve, and operator methods removed entirely (not overridden)
- ERC-5192 `locked()` always returns true

## Key Engineering Decisions

| Decision | Rationale |
|----------|-----------|
| XOR with contract address | Makes ID unique per-contract, not just per-owner globally |
| Remove transfer/approve entirely | Prevents accidental bypass vs. revert-on-call override |
| Extend ERC-721 Metadata & ERC-5192 | Well-established interfaces; wallets and indexers support them natively, compatibility is free |

## Links & Archives

- **GitHub**: <a href="https://github.com/mnusurov/erc-address-derived-sbt" target="_blank" rel="noopener noreferrer">erc-address-derived-sbt</a>
- **ERC Discussion**: <a href="https://ethereum-magicians.org/t/erc-idea-address-derived-non-transferable-token-soulbound-token/28475" target="_blank" rel="noopener noreferrer">Ethereum Magicians</a>
