
![Shell](https://github.com/code-423n4/2023-11-canto/assets/125544245/4820f530-e083-452d-a471-8d65f92be9b1)


| Contract                   | SLOC | Purpose                                           | Libraries used      |
|----------------------------|------|---------------------------------------------------|----------------------|
| [Ocean.sol ](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol)                 | 561  | The accounting engine of the shell protocol        | @openzeppelin/*     |
| [Curve2PoolAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol)      | 139  | Adapter that enables integration with the curve 2 pool | @openzeppelin/*     |
| [CurveTricryptoAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol)  | 199  | Adapter that enables integration with the curve tricrypto pool | @openzeppelin/*     |
| [OceanAdapter.sol ](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol)          | 94   | Helper contract for the adapters                   | @openzeppelin/*     |

## Overall architecture of the Ocean protocol

**Core Components**

- `Ocean.sol` - Main protocol contract that handles all key logic
- `OceanERC1155.sol` - Custom ERC-1155 ledger for accounting
- `Interactions.sol` - Defines types of interoperability interactions 
- `IOceanPrimitive.sol` - Interface for primitive contracts
- `IOceanInteractions.sol` - User-facing interaction interface

**Interaction Execution**

The `Ocean` contract contains the main logic for managing end-to-end interactions. This includes:

- Unpacking encoded interaction call data 
- Performing access control checks
- Making external adapter contract calls
- Handling asset wrap/unwrap
- Managing OceanERC1155 state updates  
- Ensuring balanced accounting throughout

**Adapter Integration** 

- `OceanAdapter.sol` provides a standard interface and utilities for connecting external protocols as Ocean primitives via adapters
- Adapters like `Curve2PoolAdapter` inherit this base functionality
- Override key functions like `wrapToken`, `unwrapToken` etc. to bridge protocols into Ocean
- Expose external liquidity pools, AMMs etc for composability 

**Managing State**

- `OceanERC1155` ledger maintains mapping from oceanIds to user balances
- Handles minting fungible/non-fungible assets during wrap
- Burns assets during unwrapping flows
- Provides standard ERC-1155 functionality alongside 

## My Analysis Approach

- Thoroughly read all contract code and comments to form a mental model 
- Reverse engineer an architectural diagram depicting components and flows
- Enumerate key design assumptions, trust boundaries and risk profiles
- Deconstruct mechanisms into functions and assess interplay 
- Analyze control flow, data flow, failure modes, economic incentives
- Identify central points of privilege and bottleneck resources
- Catalog external dependencies and quantify failure likelihoods 
- Develop attack narratives based on critical invariants 
- Synthesize findings into risk categories and mitigation strategies


## Architecture Review

- Modular framework enables flexible composition of primitives
- Relies on non-upgradable core to manage accounting and interactions 
- Balances kept in ERC-1155 with custom extensions for composability
- Adapters connect external protocols like Curve into the framework
- Carefully tracks state across nested calls to ensure consistency

**Modular Framework**

- The Ocean core implements a modular framework for composable DeFi primitives
- Interactions are predefined message types targeting external contracts
- These can wrap/unwrap tokens, compute swaps etc. through adapters  
- Framework handles all accounting/balancing during execution
- Enables building interoperable primitives without new contracts

**Core Accounting** 

- Balances tracked in custom ERC-1155 contract OceanERC1155
- Extends OpenZeppelin reference implementation
- Manages ledger updates from interactions
- Key functions: _mint, _burn, _doInteraction
- Stores state in contract storage for consistency 

**Composable Balances**

- OceanERC1155 issues fungible/non-fungible tokens
- Tokens registered to primitive contracts 
- User balances denote shares in underlying protocols
- Can be freely composed without external approvals
- Enables capital efficiency unlike standalone tokens

**Modular Adapters**

- Adapters integrate external protocols into framework
- Implement interface IOceanPrimitive 
- Wrap target protocol deposits into registered tokens
- Expose liquidity to other Ocean primitives
- Decouple core protocol from business logic  

**Ensuring Consistency**

- Careful tracking of state/balances across composed calls
- Balance ledger updated post-callback if no revert
- Push model for transfers avoids reentrancy risks
- Requires rigorous adapter review for robustness

## Centralization Risks  

- Owner can update unwrap fee without timelock (changeUnwrapFee)
- Non-transferrable ownership presents availability risks
- No checkpoints to recover in disastrous scenarios

**The Owner Privileges**

The Ocean contract grants significant, unchecked privileges to the owner address which creates centralization risks:

- Owner can update the unwrapping fee without any timelock or oversight via [changeUnwrapFee](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L196-L201). This could be abused to extract value from users.

- Ownership is non-transferrable. The owner address is the only one able to exercise privileged functions. This presents availability and maintenance risks if the key is lost or compromised.

- No recovery mechanisms are baked into the core protocol. There are no checkpoints or backstops that could reconstitute Ocean state in disastrous scenarios like critical bugs or ownership key loss.

**Mitigations** 

- Institute a reasonable timelock like 2 weeks for unwrapping fee changes

- Build an ownership transfer mechanism to decentralize control. Require timelock and staged transfers. 

- Implement privileged roles using a multi-signature scheme rather than solo owner.

- Add a pause mechanism to freeze critical logic if bugs are found.

- Explore decentralized checkpoint schemes to enable state recovery if needed.

## Mechanism Review

- Wrap flows well protected but unwrap fee math tricky 
- Reentrancy mitigations removed in key areas
- Reliance on callbacks creates need for rigorous validation

**Wrap Flows**

- The wrap flows for ERC20/721/1155 tokens seem well protected
- Transfers are initiated prior to internal balance updates
- This protects against reentrancy during wraps
- Push model avoids over withdrawal bugs

**Unwrap Fee Math**  

- The unwrapping fee logic relies on intricate decimal math
- Integer division leads to tricky rounding semantics
- Overflows or decimal issues could disrupt expected fee percentages
- Careful review of [_determineTransferAmount](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L1068-L1109) and related functions needed

**Reentrancy Surface**

- V3 removed reentrancy guards in critical [doInteraction](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L210-L218) and [doMultipleInteractions](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L229-L245)
- This was to enable adapter interactions with external protocols  
- But it dramatically increases reentrancy attack surface
- Malicious callbacks could corrupt expected balance updates

**Callbacks**

- Ocean relies heavily on reverting callbacks for safety checks
- For example, deciding whether to accept/reject token transfers
- Bugs or exceptions in callback handlers could bypass checks
- So rigorous validation of callback code is essential

## Systemic Risks

**Malicious Tokens/Adapters**

- Bugs or exploits in wrapped tokens/adapters have systemic impacts 
- A malicious ERC20 could corrupt Ocean's internal balances
- Compromised adapters are very high risk due to privileges 
- Failures not isolated due to composability

**Interleaved Call Risks**

- Complex interleaved calls between adapters
- Assumptions on call context could be violated
- For example reentrancy changes in v3
- Hard to reason about risks from unanticipated call sequences

**Lack of Upgradeability** 

- Ocean has no owner controls for upgrading logic
- Limitations of non-upgradeable architecture
- Inability to fix issues without governance process
- Freezes the current Bug Bounty terms permanently
- No flexibility for future-proofing as landscape evolves

Impacts across protocol from malicious tokens or adapters

Cascading failures if interleaved call assumptions violated

Lack of upgradeability limits ability to fix issues

**Mitigations**

- Formal verification of core protocol
- Rigorous audits for new adapters 
- Process to rapidly disable compromised adapters
- On-chain governance scheme for broad protocol changes

## Callbacks

**Safety Through Reverting Callbacks**

- The Ocean protocol relies heavily on callbacks that are expected to revert when invalid states are detected.  

- A prime example is the checks on whether to accept/reject incoming token transfers during wrapping flows. The Ocean calls:

```
ERC1155.safeTransferFrom(userAddress, Ocean, ...) 
ERC721.safeTransferFrom(userAddress, Ocean, ...)
```

- It expects these to revert if the token contract detects an issue with the transfer. For example, if the user does not actually own that NFT or token balance.

**Risks from Callback Failures** 

- However, any bugs or unexpected exceptions in these callback handlers could lead to disabled or bypassable checks. 

- For example, an ERC721 may have a bug that fails to validate ownership correctly and does not revert when expected. This would result in minting invalid Ocean balance for the user.

- Similarly exception conditions may lead to disabled validation logic.

**Mitigations**

- Every callback handler and integration point needs to undergo rigorous code review, fuzz testing and edge case enumeration before reliance.

- Critical callback code should have formal verification if possible.

- Wrapped token contracts must be evaluated for risks of buggy/exception-prone transfer logic.

## Key Components

- Ocean Core Contract - Implements protocol logic for managing interactions, accounting etc. 

- OceanERC1155 Ledger - Custom ERC-1155 contract to track user balances 

- Adapters - Bridge external protocols into Ocean framework

- Primitives - External protocols like AMMs exposed through adapters

**Wrapping Assets**

1. User calls Ocean contract to wrap an external asset (e.g. ERC20 token)

2. Ocean initiates a pull-payment style transfer from user's wallet 

3. External token contract transfers tokens to Ocean

4. Ocean mints corresponding balances to user's account in OceanERC1155 ledger

**Executing Interactions** 

1. User calls Ocean contract specifying an interaction (e.g. swap via adapter)

2. Ocean unpacks interaction details and makes external adapter call

3. Adapter executes transaction on target primitive (e.g. Curve) 

4. Ocean updates user's balances based on interaction output

**Unwrapping Assets**

1. User calls Ocean to unwrap their wrapped balance 

2. Ocean burns user's balance from OceanERC1155 ledger

3. Ocean pushes unwrapped amount back to user's wallet

4. User now has external asset balance again

## Analysis of potential issues with each core component.

**Ocean.sol**

- Removal of reentrancy guards increases attack surface area
- Relies heavily on revert checks for security - bugs in integrated callback code would bypass
- Decimal math bugs around unwrap fee could lock user funds
- Lack of upgradeability limits ability to fix issues
- Overflow risks with certain usage intensity 

**OceanERC1155.sol**

- Inherits risks from OpenZeppelin ERC-1155 reference implementation
- Should be formally verified to ensure correctness 
- Token ID generation process must be free of collisions
- Requires rigorous access control to privileged functions
- Checks for caller should validate execution context

**Interactions.sol**

- Additional interaction types increases scope for issues
- Encoding and decoding of parameters should be thoroughly validated
- SpecifiedAmount interpretation relies on code understanding semantics 

**IOceanPrimitive.sol**

- No issues directly but critical adapter code inherits risks
- Adapters have high privilege so strong review needed

**IOceanInteractions.sol**

- Function signatures could be exposed by compromised adapters
- front-running, flash loan risks if not mitigated

**Overall**

- Composability increases complexity and attack surface 
- Entire ecosystem impactable by weaknesses in any adapter
- Economic incentives should encourage virtuous cycles

### Time spent:
41 hours