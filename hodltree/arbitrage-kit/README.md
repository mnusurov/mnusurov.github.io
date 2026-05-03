# Arbitrage Kit

**Company**: HodlTree  
**Role**: Senior Solidity Developer  
**Stack**: Solidity 0.8.0 · Truffle · Ganache · Mocha · Web3.js · OpenZeppelin  
**Period**: 09/2020 - 05/2021

## Overview

Smart contract framework orchestrating atomic arbitrage trades and flash loan interactions across multiple decentralized exchanges. The architecture abstracts protocol-specific complexity through modular connector contracts, enabling seamless execution of sophisticated multi-leg trading strategies across Ethereum and Polygon. Implements delegatecall-based proxy pattern with stateful memory threading to execute complex arbitrage paths atomically without intermediate state mutations. Operated alongside a self-hosted Geth full node for low-latency RPC access in time-sensitive arbitrage execution.

## Technical Scope

### Multi-DEX Protocol Abstraction

- Unified connector interface for six major DEXs: Uniswap (V2 & V3), Balancer (V1 & V2), Curve (V1 & V2), Bancor, and 0x Protocol
- Protocol-agnostic swap orchestration via modular connector contracts
- Single atomic transaction execution across fragmented liquidity pools
- Direct pool interaction bypassing router overhead for reduced hop count and implicit fees
- **Connector-Specific Optimizations**:
  - Uniswap V3: Direct swap callbacks with tick-based slippage calculations
  - Uniswap V2: Constant-product formula (x*y=k) with native 0.3% fee handling
  - Curve: StableSwap formula for stablecoin pairs + CryptoSwap (V2) with underlying token support
  - Balancer: Vault abstraction for batch swaps and internal balance management
  - Bancor: Path-based token routing through relay networks
  - 0x: Order structure parsing and protocol fee relay integration

### Flash Loan Framework

- Flash loan initiation with callback-based repayment verification
- Integrated callback authorization via stateful memory verification (prevents reentrancy attacks)
- Support for multiple callback patterns: Uniswap V3 direct pool callbacks, Balancer Vault callbacks, and generic liquidity provider callbacks
- Automatic fee calculation and settlement within same transaction

### Stateful Execution Pipeline

- ArbMemory contract provides persistent cross-call storage for uint256 and address values
- getId/setId parameter pattern reduces calldata overhead and enables reusable transaction templates
- Delegatecall-based spell orchestration ensures atomic execution without intermediate state mutations
- Callback authorization via max uint256 key prevents unauthorized contract invocation
- Memory-based state threading with auto-cleanup semantics (delete on read) to prevent stale data leaks

### Precise Swap Control

- Exact-input and exact-output modes with explicit slippage protection
- Automatic ETH ↔ WETH conversion with minimal code duplication
- Output amount verification and minimum received constraints per swap
- Protocol-specific fee handling (Uniswap 0.3%, Curve dynamics, Bancor relay networks)

### Delegatecall-Based Proxy Architecture

- ArbProxy uses assembly-level delegatecall to execute connector logic atomically, preserving contract storage and balance state across sequential operations
- Eliminates wrapper contracts and re-entrancy concerns

### Gas Optimizations

- Minimal ERC-20 interface reduces bytecode by ~30% vs. full standard
- Direct pool interaction bypasses router overhead
- Dynamic allowance reuse (approve to max uint256) minimizes state write redundancy
- Inline assembly reduces function call overhead

## Key Engineering Decisions

| Decision | Rationale |
|----------|-----------|
| Delegatecall proxy over wrapper contracts | Atomic execution within single contract context eliminates intermediate transfers and reentrancy risks |
| Modular connectors per DEX | Protocol-specific optimizations (Curve dynamics, Bancor paths, 0x fees) reduce generic overhead |
| ArbMemory for state threading | Reduces calldata overhead and enables reusable transaction templates for complex multi-leg arbitrage |
| Flash loan callback patterns | Callback-based repayment verification enables 0-capital arbitrage on both Uniswap V3 and Balancer pools |
| Direct pool interaction | Bypasses router layers, reducing hop count and implicit protocol fees for maximum spread capture |

## Links & Archives

- **GitHub**: Not public (company policy)
- **Contracts Deployed**: Not public (company policy)

## Technical Insights

- **Flash loan callbacks authorized via a one-time token.** When a flash loan is initiated, the expected caller's address is written into a dedicated memory slot. The callback reads that slot to verify the caller - and the read destroys the value. Authorization is consumed by the act of checking it, so no explicit cleanup is needed and replay is structurally impossible.

- **Same interface for simple and chained swaps.** Every connector function accepts an optional memory reference alongside the explicit amount. When the reference is absent, the value travels directly in the call; when present, it is fetched from shared memory and erased after use. This lets the same function handle both a standalone swap and a step inside a multi-hop arbitrage path without any conditional logic in the connector itself.

- **Error messages survive proxy indirection.** Connector logic runs inside the proxy via low-level delegation, which normally discards revert reasons. The proxy captures the raw failure data and re-throws it verbatim, so debugging a failed arbitrage spell shows the original connector error rather than a generic revert - a deliberate tradeoff of assembly complexity for operational transparency.

- **Vanity address generator as companion tooling.** CREATE3Factory enables deploying contracts to deterministic addresses regardless of nonce or bytecode — useful for predictable protocol addresses across chains. To generate candidates efficiently, built a companion vanity address generator in C++/OpenCL (GPU-accelerated) and a multithreaded Node.js version, both brute-forcing CREATE3 salts until the output address matched a desired prefix pattern.

See [HodlTree page](../README.md) for additional context.