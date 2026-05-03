# reBaked DAO

**Company**: Merehead  
**Role**: Senior Solidity Developer  
**Stack**: Solidity · ERC-20 · AccessControl · ReentrancyGuard · Truffle  
**Period**: 06/2021 - 12/2021

## Overview

Architected and implemented a decentralized project management and payment protocol enabling transparent, milestone-driven collaboration between initiators, contributors, and observers. The system enforces payment terms on-chain, eliminating intermediaries and payment disputes in remote, project-based work.

## Technical Scope

### Project & Package Lifecycle
- Full on-chain workflow: proposal → approval → package creation → contributor assignment → payment → finalization
- Timestamped state transitions tracking each milestone
- Batch operations for gas efficiency: `approveProjects()`, `createPackages()`, `getMgps()`, `getBonuses()`, `getObserverFees()`

### Dual Payment Model
- **Minimum Guaranteed Payment (MGP)**: Base compensation for assigned work
- **Performance Bonuses**: Distributed proportionally via PPM (parts per million) scoring
- Fee resolution: `bonusScore / 1e6` ratio for fair bonus distribution
- Cascading budget enforcement across project → package → collaborator levels

### Flexible Token Support
- Projects operate with any ERC-20 token
- On-demand `TokenFactory` for deploying fresh `IOUToken` contracts
- Multi-asset project pools supporting diverse token economics

### Multi-Actor Role System
- **DAO Owner**: Protocol governance and configuration
- **Project Initiator**: Allocate budgets, approve work, finalize projects
- **Collaborators**: Execute work, claim MGP + bonuses permissionlessly
- **Observers**: View project state, claim observer fees
- Enforced access control on all state-mutating functions

## Key Engineering Decisions

| Decision | Rationale |
|----------|-----------|
| Library-based contract architecture | Business logic in ProjectLibrary, PackageLibrary, CollaboratorLibrary enables independent testing and reuse |
| Collision-resistant ID generation | `keccak256(msg.sender + blockhash(block.number - 1) + nonce)` provides unpredictable IDs without counters, reduces front-running |
| Triple-nested mapping state | `collaboratorData[projectId][packageId][address]` enables O(1) lookups and clean deletion across thousands of concurrent projects |
| Reentrancy-safe payment claims | All six claim functions guarded with `nonReentrant` modifier + `SafeERC20` transfers |
| Cascading budget validation | Rejecting a collaborator auto-refunds their MGP allocation back up the chain, preventing budget leaks |

## Technical Insights

- **Libraries mutate storage directly via reference, not return values.** `ProjectLibrary`, `PackageLibrary`, and `CollaboratorLibrary` receive storage pointers to their respective structs. Mutations are applied in-place inside the library, keeping the main `ReBakedDAO` contract a thin router with no logic of its own - each library can be tested independently against isolated storage fixtures.

- **Cascading budget refund is automatic, not explicit.** When a collaborator is rejected, their MGP allocation is returned up the full hierarchy: collaborator slot cleared → package budget restored → project budget incremented. This happens in one call with no separate cleanup transaction, preventing budget fragmentation across hundreds of concurrent packages.

## Links & Archives

- **GitHub**: Not public (company policy)
- **Contracts Deployed**: Prototype - not deployed to mainnet

See [Merehead page](../README.md) for additional context.
