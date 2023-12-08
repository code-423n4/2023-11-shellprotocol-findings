## Overview

The Ocean protocol aims to serve as a flexible foundation for composable DeFi primitives using a single ledger and unified token standard. The core contracts include the main Ocean engine for managing interactions and accounting, ERC-1155 tokenization, and various adapter implementations.

Overall the protocol exhibits well-factored modular design and a novel approach to interoperability and gas optimization. However, the increased flexibility and removal of constraints around reentrancy, permissions, and state management in v3 opens new potential risk vectors that should be carefully assessed.

## Approach Taken

My analysis evaluated code quality, documentation, functional correctness, security assumptions, risk analysis, adherence to specifications, extensibility, and customizability.

Findings are categorized by severity and tagged for ease of triage by developers. Recommendations in the Mitigations section aim to spur discussion rather than serve as prescriptions.

## Components

**Ocean Core Engine**

This serves as the hub for executing interactions, managing accounting, and integrating with external contracts. Key capabilities:

- Process `doInteraction` and `doMultipleInteractions` transaction batches
- Handle all token wrapping and unwrapping
- Connect adapters to underlying DeFi primitives
- Maintain core account balances and transaction state

**Tokenization (OceanERC1155)** 

The built-in ERC-1155 implementation handles token minting, burning, and transfers. It:

- Maps external assets like ERC-20s into registered Ocean tokens
- Provides balance tracking and escrow for wrapped tokens  
- Exposes transfer functionality for liquidity pools 

**Adapters**

These bridge external protocols like Curve by adapting their interfaces into normalized Ocean interactions. Adapters:

- Abstract protocol complexity into a consistent format
- Manage approvals and asset transfers  
- Perform swap calculations and interactions

**Primitives** 

The target DeFi protocols integrated with Ocean like lending pools, AMMs etc.

## Workflows

**Token Wrapping**

1. User requests ERC-20 wrap via `doInteraction()` 
2. Ocean core engine handles token transfer  
3. OceanERC1155 mints new wrapped tokens to user

**Swapping**

1. User requests swap via adapter's `primitiveOutputAmount` 
2. Adapter unwraps tokens, calls external exchange, wraps output
3. Output returned to user minus fees

**Liquidity Pools**

1. Deposits/withdrawals made through adapter to external pool
2. Adapter mints/burns internal liquidity tokens to user
3. Pool tokens tradable via OceanERC1155 transfer functions

[**Ocean.sol**](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol)

`_doInteraction()`
- Lack of validation on token relationships could allow mismatched asset transfers
-arrays passed in calldata not bounds checked (interactions, ids)

`_executeInteraction()`
- Missing rounding error tracking could lead to sustained token discrepancies
- No overflow protection on state changes and math operations 

`_getSpecifiedToken()`
- Overly complex conditional logic leads to potential execution inconsistencies
- No validation that returned token matches interaction's intent

**OceanERC1155**

`safeTransferFrom()` 
- Overreliance on selector return values violates ERC-1155 expectations
- Allowing unvetted contracts to receive untrusted tokens is unsafe

`_mintBatch()`
- No check that arrays are identical length allows balance corruption

[**CurveAdapter** ](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol)

`_convertDecimals()`
- Imprecise decimal conversions could compound minor discrepancies over time

`_determineComputeType()`
- Permitting unspecified token types can lead to unintended behavior
- Business logic gaps around computing deposit amounts

**Mitigations**

- Require interface adherence and owner signoff to integrate contracts
- Introduce custom errors for granular exception handling
- Expand test coverage of edge cases and failures
- Use an automated prover like the Solidity Analyzer toolbox






## Code Quality

- Well commented with clear explanations of logic flow  
- Explicit validation around balances and transfers
- Lighter weight adapters facilitate primitive innovation  
- Scope for improvement inConsistency, simplification, DRY principles

## Architecture


## High-Level Flow

The Ocean protocol facilitates decentralized finance interactions between users and external DeFi primitives by wrapping assets, executing computed exchanges via adapters, and maintaining a unified ledger.

## Key Steps

1. **User submits transactions** 
   - Batch of interactions via `doInteraction()` or `doMultipleInteractions()`
   - Includes details like asset being deposited, compute operation type, etc.

2. **Core engine unpacks details**
   - Parse interaction type, external contracts, and specified tokens
   - Validate expected relationships and amounts
   - Handle wrapping/unwrapping of native assets

3. **Adapter processes details** 
   - For compute interactions, adapter invoked to facilitate operation
   - Adapters customize and translate specifics for target primitive
   - Manage approvals, transfers, technical integration

4. **Primitive executes operation**
   - Target protocol like Curve, loan pool etc executes business logic 
   - Compute asset exchange, provide lending terms
   - Output returned to adapter

5. **Adapter normalizes output**
   - Handle result processing, wrapping output assets
   - Convert to standardized Ocean interaction output
   - Return net asset amounts to user

6. **Ocean logs internal state changes**
   - Mint/burn output tokens to user 
   - Adjust balances, record transfers
   - Persist transaction logs and events

This architecture supports modular integration of external DeFi protocols using adapters while maintaining a unified ledger and token standard via Ocean's backend.

**Strengths**  

- Separates core account, interaction, and integration logic  
- Abstracts complexity into reusable standards like ERC-1155
- Adapters bridge external protocols like Curve without custom coding
  
**Extensions**

- Support fee distribution to multiple stakeholders
- Allow adapters to handle native assets like ETH
- Improve custom error handling

## Security

**Severity Levels**

![Severity Legend](https://i.ibb.co/BVN2PXv/Severity-Legend.png)

| Issue | Severity | Detection |
|-|-|-|
| [Reentrancy in Adapters](#reentrancy-in-adapters) | High | Dynamic Analysis | 
| [Centralization Risks](#centralization-risks) | Medium | Architecture Review |
| [Input Validation Gaps](#input-validation-gaps) | Medium | Static Analysis |

### Reentrancy in Adapters
<a id="reentrancy-in-adapters"></a>

**Impact:** Loss of funds, arbitrary state changes  

Removing reentrancy guards allows attackers to initiate recursive calls back into adapters before state is committed.

**Mitigations:**
- Reintroduce reentrancy guards  
- Use mutex locking mechanisms
- Rigorously identify all external calls  

### Centralization Risks 
<a id="centralization-risks"></a>

**Impact:** Unauthorized modification of critical parameters

Owner role in core contracts enable unilateral control of fee rates, pause status, and other sensitivie functions.

**Mitigations:**

- Timelock or multi-sig for sensitive owner functions 
- Break up roles into fee manager, pauser, etc  

### Input Validation Gaps
<a id="input-validation-gaps"></a>

**Impact:** Unexpected behavior, contract exploits  

Lack of validation around key assumptions could allow attacks like force wrapping unowned tokens.

**Mitigations:**

- Comprehensively validate inputs adhere to documented requirements and invariants  
- Use a robust property-based testing framework like Foundry to mass generate test cases



### Time spent:
36 hours