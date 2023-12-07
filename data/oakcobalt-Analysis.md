## Summary

### Description:

Shell Protocol is a collection of Ethereum Virtual Machine (EVM)-based smart contracts deployed on the Arbitrum One blockchain. Unlike traditional DeFi protocols that use single-purpose smart contracts, Shell stands out as a hub for a modular ecosystem of services. Its unique design facilitates the bundling of multiple smart contracts or the creation of new ones, providing users with a streamlined process to combine various services within a single transaction.

**Existing Standards:**

- The protocol adheres to conventional Solidity and Ethereum practices, primarily utilizing the ERC20, ERC721 and ERC1155 standards.
- The protocol also uses ERC165 standard for Ocean.sol for introspection.

## 1- Approach:

- Scope: The scope is limited to Ocean.sol and the adapter implementations. But extra attention is given also to LiquidityPool.sol, and BalanceDelta.sol for a more inclusive review of various use cases related to Ocean.sol and adapters.
- Roles: Main focus of user flows concerning the Ocean owner, general user that add liquidity, remove liquidity, swap or wrap tokens, or primitive/adapter creators.

## 2- Centralization Risks:

Overall low centralization risks given the limited number of access-controlled or admin-controlled functions.

### Ocean.sol

- Owner is able to set fee division factor, which is a Dao-controlled process. This is low risk in centralization.
- Note that the owner flows are currently inadequately supported in Ocean.sol. Key owner flow such as fee claim and redistribution needs to go through general user methods such as `_erc20unwrap()`. This is good for decentralization but impedes the efficiency of owner fee management, and also will trap part of owner entitled fees in the contract. See my H/M submission for more information.

## 3- Systemic Risks:

Overall low systemic risks given the token exchange in Ocean.sol will essentially take place between primitive contracts and user accounts. Ocean.sol isn’t directly exposed to loss or gain from the user or primitive.

Slippage is also taken care of in Ocean liquidity pool or Ocean adapter contracts.

### Counterparty Risks:

There are increased counterparty risks from malicious primitives (external liquidity pools, adapter contracts, or Ocean native liquidity pools). 

Although the impact is contained between primitive and users, Ocean has a small exposure to such risks in the fee management and redistribution process. 

Whenever a malicious token is wrapped in Ocean.sol, part of the fee is collected in the malicious token format and wrapped in Ocean id for the owner. Depending on the exact process of the ocean owner’s fee redistribution, a malicious token might DOS the entire fee unwrap and redistribution process.

## 4- Mechanism Review:

### Inconsistent interaction orders:

Ocean.sol ‘s `doMultipleInteractions()` might require different interaction orders even for the same token pairs. This can be quite confusing to the general users to know which order is the correct order of interactions to pass to `doMultipleInteractions()`. This compromises the user experience.

The proper interaction orders depend on the implementation of the primitive contract and might not be known or fully understood by the user ahead of time. 

Suppose this example:

A user wants to add liquidity by depositing TokenA through `computeOutputAmount` method. The user doesn’t have any token wrapped in Ocean. The user invokes `doMultiplerInteractions()`.

**(1) Case 1: The primitive is native ocean LiquidityPool that registered LP tokens.**

Valid interaction order A: computeOutputAmount() → wrapErc20(). Since all token mints and burnt are taken care of at the end of `doMultiplerInteractions()`. Ocean.sol will directly mint input tokens (TokenA) for the primitive ahead of time, and user’s input token can be wrapped afterwards.

Valid interaction order B: wrapErc20() → computeOutputAmount(). The reverse order will also work for the same reasons above.

**(2) Case 2: The primitive is an ocean adapter wrapping an external liquidity pool.**

Valid interaction order B: wrapErc20() → computeOutputAmount(). This is the only valid interaction order in this case, because if Ocean.sol doesn’t have any wrapped TokenA ahead of time, the ocean adapter will not receive and unwrap any TokenA input token to be sent to the external liquidity pool. In this case the reverse order will cause revert.

### **No On-Chain Mechanism for Fee Redistribution:**

In Ocean.sol, fees are minted for various wrapped tokens to owner(). However, this information can only be tracked and calculated off-chain through event listing of minting, consider on-chain implementation for queries on all tokens owned by Ocean owner.

### **`computeInputAmount` doesn't need to be disabled in OceanAdapter.sol**

In OceanAdapter.sol, the abstract generalized adapter, `computeInputAmount` is implemented with a direct `revert()`. Although this `revert()` can be overwritten in deriving contracts,  this is still unnecessary and potentially misleading.

(1) OceanAdapter.sol is an abstract contract and can be overwritten. Consider refactoring `revert()` implementation in Curve2PoolAdapter and CurveTricryptoAdapter. The abstract contract is better remained as a neutral contract unbiased of any specific swapping or liquidity management methods which might differ based on external protocols.

(2) `computeIntputAmount` can still be supported for ocean adapters if the external liquidity provides a compatible method. However, the issue is Ocean.sol doesn't implement `_computeInputAmount` interaction correctly. In Ocean.sol `_computeInputAmount()`, the order in which the primitive's balance of output token is decreased before a call to the primitive's `computInputAmount()` is problematic for any ocean adapters. Ocean.sol `computInputAmount()`'s implementation needs to be revised to allow ocean adapters' `computeInputAmount` to properly work. See my H/M submission for details.

### **Curve2PoolAdapter will not work with all Curve 2Pools:**

Some curve 2 pools have a separate LpToken contract which is different from the liquidity pool contract. These pools will not work with Curve2PoolAdapter. See my QA submission for details.

For example, Curve 2 pools on Arbitrum are generally compatible with Curve2PoolAdapter. See below.
![Screenshot 2023-12-07 at 1 10 26 PM](https://github.com/cowri/ocean/assets/138168196/1e531987-d6d7-4d1b-9d99-be668b3d9fa8)
(https://www.geckoterminal.com/arbitrum/curve_arbitrum/pools)

However, Curve 2 pools on Mainnet are generally incompatible. See below.
![Untitled](https://github.com/cowri/ocean/assets/138168196/258b5ae4-60a1-4d02-b3b2-2fa21e49b8af)
(https://www.geckoterminal.com/eth/curve/pools)

It should also be noted that, generally curve 2 pools on arbitrum are compatible but it doesn’t prevent any future curve 2 pool deployment or other derivative implementation on arbitrum to become incompatible.

### OceanAdapter currently doesn’t support some mainstream swapping and liquidity management protocols:

OceanAdapter.sol is intended to be generalized abstract contract that allows protocol and pool-specific implementation to inherit and overwrite. However, OceanAdapter currently is not compatible with a few common swapping or liquidity management methods such as two-sided liquidity adding, two-sided liquidity removing, ERC721 liquidity position minting, etc. Any external liquidity pools that adopt the above-mentioned methods will be incompatible. This include Uniswap V2 / V3.
**(1) Two-sided liquidity adding:** 

This requires two input tokens and one output token in one function call. Currently not supported.

**(2) Two-sided liquidity removing:**

This requires one input token and two output tokens in one function call. Currently not supported.

**(3) ERC721 liquidity position minting:**

This requires OceanAdapter to support receiving ERC721 tokens. Currently, OceanAdapter only supports receiving ERC1155. 

**(4) ERC721 liquidity position removing:**

See reason for (3).



### Time spent:
30 hours