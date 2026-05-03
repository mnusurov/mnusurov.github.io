# HodlSwap

**Company**: HodlTree  
**Role**: Senior Solidity Developer  
**Stack**: Solidity 0.8.0 · Truffle · Ganache · Mocha · Web3.js · OpenZeppelin  
**Period**: 09/2020 - 05/2021

## Overview

Fork of Uniswap V2 that replaces its hardcoded 0.3% fee with a per-pair fee-tier system, adds a flexible protocol fee model, and introduces targeted gas optimizations across the swap and liquidity paths. Multi-hop routing is redesigned with a bytes-encoded path format (adapted from Uniswap V3) to carry the fee tier per hop inline, enabling each leg of a multi-hop swap to route through a different fee-tier pool.

## Changes vs Uniswap V2

### Per-Pair Configurable Swap Fees

Uniswap V2 hardcodes 0.3% across every pool in the protocol. HodlSwap stores the swap fee inside each Pair contract, set once at deployment. Stablecoin pools can run at 0.05%, standard pools at 0.3%, exotic pairs at 1% - all within the same factory, with no governance action needed to add a new tier.

### Multiple Pools per Token Pair

Uniswap V2 enforces one pool per token pair via a two-dimensional mapping. HodlSwap extends this to three dimensions by adding fee as a key, so the same token pair can have multiple concurrent pools at different fee levels. Both `createPair` and `getPair` require the fee tier to identify the correct pool.

### Flexible Per-Pair Protocol Fee

Uniswap V2 has a single global protocol fee switch: either all pools send 1/6 of swap fees to a `feeTo` address, or none do.

HodlSwap stores a protocol fee fraction independently per pair. The fraction is expressed as a divisor - 1/2, 1/3, 1/5, or off - so different pool types can contribute different shares of revenue without affecting each other. The factory sets a default applied at pool creation and can update individual pools independently.

### Fee-Aware Deterministic Addressing

Because the same token pair can now exist at multiple fee tiers, the CREATE2 salt used to deploy each pool includes the fee alongside the token addresses. This preserves deterministic off-chain pair address calculation while ensuring each fee-tier pool has a unique, predictable address - no registry lookup required.

### Bytes-Encoded Multi-Hop Path (adapted from Uniswap V3)

Uniswap V2 Router encodes multi-hop routes as a plain array of token addresses. With a global fee this is sufficient, but per-pair fees require knowing which fee tier to use at each hop. HodlSwap adopts the Uniswap V3 approach: the path is a packed byte sequence alternating token addresses and 2-byte fee values. Each swap function decodes the path on the fly, locating the correct pool at each hop without additional calldata or external lookups.

## Gas Optimizations vs Uniswap V2

The fee-tier architecture required rethinking several storage and computation patterns. The changes below reduce gas across every core operation.

### TWAP Oracle Removed (~400–600 gas per swap)

Uniswap V2 updates two price accumulator variables on every swap to support its on-chain TWAP oracle. This requires reading the current timestamp, computing a time-weighted price, and writing two storage slots. HodlSwap removes the TWAP entirely - the `blockTimestampLast` field and accumulator updates are gone, cutting 400–600 gas from every swap.

### Per-Pair Protocol Fee Packed into the Reserves Slot (zero extra storage)

Uniswap V2 stores reserves and the last block timestamp together in one 256-bit slot: two 112-bit reserves and a 32-bit timestamp. With the timestamp removed, HodlSwap repacks the slot as two 104-bit reserves plus an 8-bit protocol fee value. Adding per-pair protocol fee configuration costs no additional storage slots - the `feeProtocol` field rides in the space freed by the removed timestamp.

### No SLOAD for Pair Addresses in Swap Hot Path (~2,000 gas per lookup)

Uniswap V2's library also computes pair addresses via CREATE2, so at the library level both protocols avoid registry lookups. HodlSwap extends this: because the fee is part of the CREATE2 salt, any fee-tier pool address is computable off-chain with the same formula, with no registry access at any call site.

### Packed Bytes Path Reduces Calldata Cost (~40 gas per hop)

Uniswap V2 encodes a multi-hop path as an array of 32-byte ABI-padded addresses - 32 bytes per token. HodlSwap packs the path as 20-byte addresses interleaved with 2-byte fees - 22 bytes per hop. For a 3-hop swap this cuts roughly 120 bytes of calldata, saving around 120–180 gas on calldata alone, plus eliminating any separate fee array.

### Assembly-Optimized Path Decoding

The `BytesLib` and `Path` libraries decode token addresses and fee values from the packed bytes path using direct memory operations in inline assembly. This avoids the type-conversion and bounds-checking overhead of equivalent Solidity, saving roughly 30–50 gas per decode call - multiplied across every hop in every swap.

### Local Caching of Per-Pair State

Because swap fees and protocol fees are now dynamic per-pair values read from storage, the Pair contract caches them in local variables at the start of each operation. A value read once from storage (2,100 gas cold, 100 gas warm) and cached costs 3 gas on every subsequent access within the same call. For operations like mint and burn that access the protocol fee multiple times, this saves several hundred gas per transaction.

## Key Engineering Decisions

| Decision | Rationale |
|---|---|
| Fee stored per pair, immutable post-deployment | No governance mechanism needed to add fee tiers; eliminates attack surface around fee changes |
| Three-dimensional pair mapping | Same token pair at different fee levels is a first-class concept, not a workaround |
| Protocol fee as a fraction divisor | More expressive than V2's binary on/off; each pool can be tuned independently |
| Fee included in CREATE2 salt | Deterministic addressing across all fee tiers without registry overhead |
| Bytes path with inline fees | Necessary to route multi-hop swaps through the correct fee-tier pool at each leg |
| TWAP removed | Saves 400–600 gas per swap; external oracle dependency is acceptable for this use case |

## Links & Archives

- **GitHub**: Not public (company policy)
- **Contracts Deployed**: Prototype - not deployed to mainnet

## Technical Insights

- **`feeProtocol = 0` means the protocol captures the maximum share, not zero.** The `int8 feeProtocol` field encodes a fraction divisor: the protocol fee is `1 / (feeProtocol + 1)` of the swap fee. A value of `0` means `1/1` - 100% of the swap fee goes to the protocol. Negative values disable the fee entirely. This encoding fits the storage layout and the mint formula elegantly, but the zero-equals-maximum semantics are the opposite of what the name suggests and require explicit documentation to avoid misconfiguration.

- **The bytes path has no typed index access to structured elements, which required an explicit reverse-traversal API.** Uniswap V2 paths are Solidity arrays: `path[path.length - 1]` directly yields a typed `address`. The bytes path supports offset-based reads - `decodeLastPool` computes `(numPools - 1) × 22` and reads from there - but there is no equivalent of a typed index. Reaching the last pool requires counting pools, computing the byte offset, then manually decoding a 20-byte address and a 2-byte fee into their respective types. These steps had to be encapsulated in `decodeLastPool` and `skipLastToken`. The calldata saving came at the cost of writing a reverse-traversal API that V2 got for free from the language.

- **Each fee tier is a fully independent pool contract, not a virtual tier within one pool.** Uniswap V3 hosts multiple fee tiers for the same token pair inside a single pool through its concentrated liquidity model. HodlSwap, as a V2 fork, deploys a separate Pair contract per fee tier - each with its own reserves, its own LP token, and its own kLast. Arbitrage between a 0.05% pool and a 0.3% pool of the same token pair requires two separate transactions against two separate contracts. Liquidity does not flow between tiers automatically.

See [HodlTree page](../README.md) for additional context.