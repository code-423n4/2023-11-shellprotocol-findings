The Shell Protocol is a set of smart contracts that run on the `EVM` and are hosted on `Arbitrum One`. Unlike other DeFi protocols that use individual, specialized smart contracts, Shell Protocol is a central hub for a variety of services. It's designed to make it easier to group several smart contracts or create new ones, and for users to carry out multiple services in a single transaction. The scope of this contest includes the smart contracts: `Ocean.sol`, `Curve2PoolAdapter.sol`, `CurveTricryptoAdapter.sol` and `OceanAdapter.sol`.

The contract `Ocean.sol` is designed to interact with various token standards like `ERC-20`, `ERC-721`, `ERC-1155`, and custom primitives through a multitoken ledger system. It allows users to perform a series of interactions (defined as `Interaction` structs) with these tokens, such as wrapping, unwrapping, and computing input/output amounts for trades. The key features of the contract are:

1. The contract allows users to wrap and unwrap `ERC-20`, `ERC-721`, and `ERC-1155` tokens into the Ocean's accounting system. Wrapped tokens are represented with a unique `oceanId` derived from the original token's contract address and token ID (if applicable).

2. The contract charges a fee for unwrapping tokens, calculated as `unwrapAmount / unwrapFeeDivisor`. The fee is rounded down, and if the `unwrapAmount` is less than `unwrapFeeDivisor`, no fee is charged. The minimum fee divisor is set to 2000, which corresponds to a fee of 0.05%.

3. ERC-20 tokens are wrapped with an 18-decimal representation to facilitate interactions between tokens with different decimal places.

4. The contract uses `_ERC1155InteractionStatus` and `_ERC721InteractionStatus` to prevent reentrancy attacks during token wrapping and unwrapping.

5. Users can execute single or multiple interactions in one transaction. The contract emits events for each interaction type, providing transparency over actions taken.

6. The contract allows approved forwarders to execute interactions on behalf of users, provided they have the necessary approvals.

7. The contract owner can change the unwrap fee divisor, which affects the fee rate for unwrapping tokens.

8. The contract implements `IERC721Receiver` and `IERC1155Receiver` to handle incoming token transfers of these types.

9. The contract can interact with external contracts (primitives) that implement custom business logic for token exchanges.

The `Curve2PoolAdapter.sol` contract is an adapter for interacting with a Curve Finance 2pool, specifically for swapping between two tokens (USDC and USDT in this case), adding liquidity, and removing liquidity from the pool. The contract inherits from `OceanAdapter` contract. Here's a breakdown of the code and its functionality:

1. `xToken`, `yToken`, and `lpTokenId` are immutable variables representing Ocean IDs for the two tokens in the Curve pool and the liquidity provider (LP) token, respectively, `indexOf` is a mapping from Ocean IDs to Curve pool indices, `decimals` is a mapping from Ocean IDs to the number of decimals for the corresponding tokens.

2. `wrapToken` and `unwrapToken` are internal functions that handle wrapping and unwrapping tokens into and out of the Ocean platform, respectively.

3. `primitiveOutputAmount` is an internal function that interacts with the Curve pool to perform swaps, add liquidity, or remove liquidity based on the `ComputeType` determined by the `_determineComputeType` function. It checks for slippage by comparing the actual output amount with the minimum expected output amount provided by the user. If the actual amount is less than the minimum expected, it reverts with `SLIPPAGE_LIMIT_EXCEEDED`. Emits corresponding events (`Swap`, `Deposit`, `Withdraw`) based on the action performed.

4. `_determineComputeType` is a private view function that determines the type of computation (swap, deposit, or withdraw) based on the input and output tokens.

The contract uses `immutable` for variables that are set once during construction, which is a good practice for gas optimization and security. The contract uses custom errors (`INVALID_COMPUTE_TYPE` and `SLIPPAGE_LIMIT_EXCEEDED`) instead of revert strings, which is also a gas optimization. The contract checks for slippage, which is an important security measure to prevent users from receiving fewer tokens than expected due to price movements.The contract approves the maximum possible token allowance to the Ocean and Curve pool. This is a common pattern but can be risky if either the Ocean or Curve pool contracts are compromised. A safer approach could be to approve only the required amount for each transaction. The contract assumes that the `primitive` address provided to the constructor is a legitimate Curve pool contract. If this address is incorrect or malicious, it could lead to loss of funds.

The third contract `CurveTricryptoAdapter.sol` acts as an adapter for interacting with the Curve Tricrypto pool, which is a liquidity pool for swapping between `USDT`, `WBTC`, and `ETH`. The key features of the contract are:

1. The contract defines immutable variables for the Ocean IDs of the tokens involved in the Curve pool (xToken, yToken, zToken) and the liquidity pool token (lpTokenId). It also maps these Ocean IDs to their corresponding indices in the Curve pool and their decimal places.

2. The `wrapToken` and `unwrapToken` functions are used to wrap and unwrap tokens into and out of the Ocean protocol, respectively. For the ETH token (zToken), it uses a special handling since ETH is native and not an ERC20 token.

4. The `primitiveOutputAmount` function is the core function that interacts with the Curve pool to perform swaps, add liquidity, or remove liquidity. It determines the type of action to perform based on the input and output tokens. It checks for slippage by ensuring the actual output amount is not less than the minimum expected output amount provided by the user. It emits events based on the type of action performed (Swap, Deposit, Withdraw).

5. The `_determineComputeType` function determines whether the user is trying to swap tokens, deposit into the pool, or withdraw from the pool based on the input and output tokens.

6. The contract includes a fallback function that allows it to receive ETH directly.

The contract `OceanAdapter.sol` is intended to be a helper contract for shell adapters within a system that involves token wrapping and unwrapping. The main functionality of this contract is the following:

1. The contract has two immutable state variables, `ocean` and `primitive`, which are addresses set at the time of contract deployment and cannot be changed afterward. These likely represent the main Ocean system contract and an external contract or system (the "primitive") that the adapter interacts with.

2. There is a mapping from a `uint256` token ID to an `address` that represents the underlying token addresses corresponding to Ocean IDs.

3. The `onlyOcean` modifier ensures that certain functions can only be called by the Ocean contract.

4. `computeOutputAmount`: This function is meant to be called by the Ocean contract to calculate the amount of output token that should be given to the user based on an input amount. It unwraps the input token, calculates a fee, computes the output amount using a function `primitiveOutputAmount` (which is not implemented in this abstract contract), and then wraps the output token.

5. `_fetchInteractionId` and `_calculateOceanId`: These internal functions generate identifiers for interactions and Ocean IDs based on token addresses and other parameters.
   
6. `onERC1155Received`: This function is a hook that is called when an ERC1155 token is received. It simply returns a magic value that signals the contract can handle ERC1155 tokens.
   
7. `getTokenSupply`: This function always returns 0, indicating that this primitive should not hold any tokens.
   
8. `_convertDecimals`: A utility function to convert amounts between different decimal bases.
   
9. `primitiveOutputAmount`, `wrapToken`, `unwrapToken`: These are internal virtual functions meant to be overridden in derived contracts to provide specific functionality for computing output amounts and handling token wrapping and unwrapping.

The protocol is well written and documented. It's been a pleasure to spend the last few days working on it!

### Time spent:
15 hours