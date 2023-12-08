## Introduction

Shell Protocol is a set of EVM-based smart contracts [on Arbitrum One](#). Unlike other DeFi protocols that rely on monolithic, single-purpose smart contracts, Shell is a hub for a modular ecosystem of services. Its design makes it much simpler to bundle several smart contracts or build new ones, and for users to batch many services in one transaction.

Shell Protocol has two major components:

- [The Ocean](#): a shared accounting system for easily composing DeFi primitives
    
- [Proteus](#): an AMM engine that can precisely replicate any bonding curve shape while using fungible LP tokens
    

## Overview

Shell v3 improves upon the fundamentals developed in Shell v2.

The goal of Shell v3 is to make the Ocean compatible with external protocols through the use of adapter primitives.

# Audit approach

1.  Read the documentation.
2.  Try to understand how the system works by looking at the docs and website
3.  Look at each code individually and focus on the internal function calls.
4.  Read the test files to get a better idea of the end-to-end scenarios.
5.  Write a Report by compiling all the insights I gained throughout the line-by-line code review.

&nbsp;

### Contracts in Scope

- [ ] **OceanAdapter.sol**: is an abstract helper contract designed to facilitate interactions between the Ocean protocol and external primitives.

&nbsp; Core Functions:

1.  - `computeOutputAmount`: Given an input token, output token, and input amount, this function calculates the output amount after unwrapping the input token and applying a fee. It then calls `primitiveOutputAmount` to determine the output amount based on the unwrapped amount and wraps the output token before returning the amount.
    - `computeInputAmount`: This function is not implemented and reverts when called. It's meant to calculate the input amount given an output amount, but its functionality is not provided in this contract.
    - `_fetchInteractionId` and `_calculateOceanId`: These internal functions generate identifiers for interactions and Ocean IDs for tokens, respectively.

- [ ] **CurveTricryptoAdapter.sol**:  is an adapter specifically designed for interacting with the Curve Tricrypto pool.

What it Does :

- It helps the Ocean protocol interact with the Curve Tricrypto pool.
- Handles actions like swapping, adding liquidity, and removing liquidity in this specific pool.

Core Functions:

- [ ] `primitiveOutputAmount`: Swaps tokens, adds or removes liquidity in the Curve pool, and checks if the output is above a minimum threshold.
    
- [ ] `wrapToken`: Converts a token into its equivalent representation in the Ocean protocol.
    
- [ ] `unwrapToken`: Converts a token from its Ocean protocol representation back to the original token.
    
- [ ] `_approveToken`: Gives permission for the Ocean protocol and Curve pool to use the contract's tokens.
    
- [ ] `_getBalance`: Gets the amount of a specific token that the contract holds.
    
- [ ] `_determineComputeType`: Decides whether the operation is a swap, deposit, or withdrawal based on the tokens involved.
    
- [ ] `fallback`: Allows the contract to receive Ether directly with no extra data.
    
- [ ] **Curve2PoolAdapter.sol**: acts as a bridge between the Ocean protocol and the Curve 2pool for the USDC-USDT pool.
    

Tokens Involved\*\*:\*\*

- `xToken`: Represents one of the tokens in the Curve pool.
- `yToken`: Represents the other token in the Curve pool.
- `lpTokenId`: Represents the Curve LP (Liquidity Provider) token.

Core functions:

- **primitiveOutputAmount**: Executes the main operations of swapping tokens, adding liquidity, or removing liquidity from the Curve 2pool. It ensures that the output amount is not less than the minimum expected to protect against slippage.
    
- **wrapToken**: Wraps an ERC20 token into its corresponding Ocean token representation. This is an internal function used before interacting with the Curve pool if the tokens need to be in the Ocean protocol format.
    
- **unwrapToken**: Unwraps an Ocean token back into its original ERC20 token. This is an internal function used after interacting with the Curve pool to convert the tokens back from the Ocean protocol format.
    

&nbsp;

- [ ] **Ocean.sol :** It is designed to interact with various token standards such as ERC-20, ERC-721, ERC-1155, and custom tokens that implement the IOceanPrimitive interface. The contract provides a framework that allows users to execute a list of interactions, which involve calls to external contracts resulting in updates to the Ocean's accounting system.

The core functions is categorized based on their primary responsibilities:

1.  **Interaction Execution:**
    
    - `doInteraction`: Executes a single interaction provided by the user.
    - `doMultipleInteractions`: Executes multiple interactions provided by the user.
    - `forwardedDoInteraction`: Executes a single interaction on behalf of another user, provided the caller is an approved forwarder.
    - `forwardedDoMultipleInteractions`: Executes multiple interactions on behalf of another user, provided the caller is an approved forwarder.
2.  **Token Wrapping and Unwrapping:**
    
    - `_erc20Wrap`: Wraps an ERC-20 token into the Ocean's ledger.
    - `_erc20Unwrap`: Unwraps an ERC-20 token from the Ocean's ledger.
    - `_erc721Wrap`: Wraps an ERC-721 token into the Ocean's ledger.
    - `_erc721Unwrap`: Unwraps an ERC-721 token from the Ocean's ledger.
    - `_erc1155Wrap`: Wraps an ERC-1155 token into the Ocean's ledger.
    - `_erc1155Unwrap`: Unwraps an ERC-1155 token from the Ocean's ledger.
    - `_etherUnwrap`: Unwraps Ether from the Ocean's ledger.
3.  **Primitive Interactions:**
    
    - `_computeOutputAmount`: Interacts with an external primitive to compute the amount of output token to be received for a given input token and amount.
    - `_computeInputAmount`: Interacts with an external primitive to compute the amount of input token required to receive a specified output token amount.
4.  **Fee Management:**
    
    - `changeUnwrapFee`: Allows the contract owner to change the fee divisor for unwrapping tokens.
5.  **ERC-1155 and ERC-721 Receiver Functions:**
    
    - `onERC721Received`: Handles the receipt of a single ERC-721 token.
    - `onERC1155Received`: Handles the receipt of a single ERC-1155 token.
    - `onERC1155BatchReceived`: Handles the receipt of multiple ERC-1155 tokens.
6.  **Support Functions:**
    
    - `supportsInterface`: Indicates whether the contract implements a specific interface.
    - `_calculateUnwrapFee`: Calculates the fee for unwrapping tokens.
    - `_grantFeeToOcean`: Grants the unwrap fee to the Ocean's owner.
7.  **Utility Functions:**
    
    - `_executeInteraction`: Core logic for executing interactions.
    - `_unpackInteractionTypeAndAddress`: Extracts the interaction type and external contract address from an interaction.
    - `_getSpecifiedToken`: Determines the specified token based on the interaction type.
    - `_increaseBalanceOfPrimitive`: Increases the balance of a primitive contract for a given token.
    - `_decreaseBalanceOfPrimitive`: Decreases the balance of a primitive contract for a given token.

&nbsp;

### lists functions that are explicitly declared public or payable

| 🌐Public | 💰Payable |
| :--- | :--- |
| 16  | 7   |

| External | Internal | Private | Pure | View |
| :--- | :--- | :--- | :--- | :--- |
| 14  | 56  | 14  | 8   | 9   |

## Test analysis

The audit scope of the contracts to be reviewed is 98%.

## Codebase Quality

- **Modular Design:** The code is organized into different contracts, each responsible for specific functionality. This makes the codebase easier to understand and maintain.
    
- **Clear Comments:** There are comments throughout the code that explain the purpose of functions, variables, and sections. This enhances readability and comprehension.
    
- **Events:** Events are used to provide transparency and enable easier tracking of important contract interactions.
    

&nbsp;

## Centralization Risk

centralization Risk are points here

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/bot-report.md#m-01

&nbsp;

## Gas Optimization

Shell Protocol is generally efficient in terms of gas optimizations, many generally accepted gas optimizations have been implemented, gas optimizations with minor effects are already mentioned in automatic finding, here is some more.

\[G-01\] Use assembly to validate `msg.sender`

\[G-02\] uint8 is not always cheaper than uint256

\[G‑03\] Using `bool`s for storage incurs overhead

\[G-04\] Use hardcode address instead `address(this)`

\[G-05\] Use constants instead of type(uintx).max

\[G-06\] Use `ERC721A` instead `ERC721`

\[G-07\] Return values from external calls can be cached to avoid unnecessary call

\[G-08\] Make 3 event parameters indexed when possible

## **Documentation**

The documentation provided for the Shell contract is quite comprehensive and detailed in terms of explaining its functionality, parameter usage, purpose, and overall architecture.

## Time Spent on the Audit

- 23 hours

&nbsp;

### Time spent:
23 hours