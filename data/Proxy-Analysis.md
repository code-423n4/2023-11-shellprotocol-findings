# Analysis Report

![ShellProtocol](https://gist.github.com/assets/72068235/a40818ae-62e3-4032-bf40-e6845e5926b9)


## Approach taken in evaluating the codebase

A top-down approach was used to understand the protocol codebase.
Focusing first on understanding what problem the protocol is trying to solve, leveraging documentation and skimming through functions in the smart contracts.
Following this foundational understanding, the getting started tips from [Ocean.sol#L61](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L61) were used to understand the main smart contract. Finally, other smart contracts in scope were read.

## Smart Contracts

The security review consisted of 4 contracts `Ocean.sol`, `OceanAdapter.sol`, `Curve2PoolAdapter.sol` and `CurveTricryptoAdapter.sol`.

The contracts are well written with good NatSpec comments, but have some redundant comments, and long functions that can be broken down into smaller ones. They follow good coding practices, except for using `assert()` instead of `require()` in certain situations. Almost all the time it is better to use `require()` because the `assert()` function generates an error of type `Panic(uint256)`. Code that works properly should never Panic, even on invalid external input.

The contracts are planned to be deployed on Arbitrum, however they use solidity version 0.8.20, which is currently unsupported on Arbitrum.

They have very little centralization risk, only in `changeUnwrapFee()` which calculates the fee paid when unwrapping tokens from the Ocean. If the owner is a simple EOA (Externally Owned Account) the address will be able to change the fee however they like, and EOA wallets are very susceptible to theft. This is why it is recommended to use a multisig wallet or even governance as the owner. Before changing the unwrap fee, the users should be noted somehow on social media or through any other platforms.

Another potential risk is that `Curve2PoolAdapter.sol` and `CurveTricryptoAdapter.sol` use infinite approvals without any means to revoke them for both Ocean and the primitive with which the user will interact. If any of the approved contracts get compromised the user could lose funds. It is highly suggested to implement some kind of revoke functionality. [Here](https://revoke.cash/exploits) we can see all the approval hacks & exploits that happened before.

### Ocean.sol

The Ocean is one of the most important contracts of the Shell Protocol, and has three functionalities.

1. Accounting of balances for DeFi primitives such as AMMs, lending pools, stablecoins or NFT markets
2. Wraps / Unwraps ERC20, ERC721 or ERC1155 tokens into ERC1155 tokens that can be used on any DeFi primitives in the Shell ecosystem
3. Tracks intermediate balance updates in memory instead of storage

Architecture diagram of the Ocean

![Architecture diagram of the Ocean](https://miro.medium.com/v2/resize:fit:1400/0*ZwcTn8hFcSGaSpZI)

From the architecture above we can see that users interact with the Ocean contract through interactions. In order to execute these interactions users call `doInteraction()` to perform one interaction or `doMultipleInteractions()` to perform multiple consecutive interactions. Intermediate balances are tracked in memory when users call `doMultipleInteractions()` saving gas.

Interactions are structures consisting of five variables.

```solidity
struct Interaction {
    bytes32 interactionTypeAndAddress;
    uint256 inputToken;
    uint256 outputToken;
    uint256 specifiedAmount;
    bytes32 metadata;
}
```

- `bytes32 interactionTypeAndAddress` consists of 1 byte representing the interaction type (explained later) and 20 bytes representing an external contract with which the user wants to interact with
  - Example to unwrap USDC: `bytes32 interactionTypeAndAddress = 0x010000000000000000000000a0b86991c6218b36c1d19d4a2e9eb0ce3606eb48`
    - `0x01` represents the interaction type `UnwrapErc20` and `0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48` represents the USDC token address

- `uint256 inputToken` is what the user provides / gives as input to the interaction

- `uint256 outputToken` is what the user gets as output of the interaction

- `uint256 specifiedAmount` is the amount of the specified token. Specified tokens are Ocean wrapped tokens.
  - if this amount is `type(uint256).max` it is a request by the user to use the intra-transaction delta of the specified token as the specified amount (used with `doMultipleInteractions()`)

- `bytes32 metadata` is either forwarded to the external contract if the interactions are Compute* or is used during wrapping / unwrapping ERC721 or ERC1155 tokens by casting to `uint256` to represent the token id of the specific token we want to use.

The interaction types are enums consisting of the following 9 types:

```solidity
enum InteractionType {
    WrapErc20,
    UnwrapErc20,
    WrapErc721,
    UnwrapErc721,
    WrapErc1155,
    UnwrapErc1155,
    ComputeInputAmount,
    ComputeOutputAmount,
    UnwrapEther
}
```

- `WrapErc20`
  - "wraps" the ERC20 token by transferring the ERC20 token from the user to the Ocean contract
  - at the end of the interaction mints the equivalent amount of ocean wrapped token to the user

- `UnwrapErc20`
  - "unwraps" the ERC20 token by transferring the ERC20 token from the Ocean contract to the user
  - at the end of the interaction burns the equivalent amount of ocean wrapped token from the user

- `WrapErc721`
  - "wraps" the ERC721 token by transferring the ERC721 token from the user to the Ocean contract
  - at the end of the interaction mints the ocean wrapped token to the user
  - it uses the `metadata` to specify which specific `tokenId` to transfer

- `UnwrapErc721`
  - "unwraps" the ERC721 token by transferring the ERC721 token from the Ocean contract to the user
  - at the end of the interaction burns the ocean wrapped token from the user
  - it uses the `metadata` to specify which specific `tokenId` to transfer

- `WrapErc1155`
  - "wraps" the ERC1155 token by transferring the ERC1155 token from the user to the Ocean contract
  - at the end of the interaction mints the ocean wrapped token to the user
  - it uses the `metadata` to specify which specific `tokenId` to transfer

- `UnwrapErc1155`
  - "unwraps" the ERC1155 token by transferring the ERC1155 token from the Ocean contract to the user
  - at the end of the interaction burns the ocean wrapped token from the user
  - it uses the `metadata` to specify which specific tokenId of the token to transfer

- `ComputeInputAmount`
  - `Ocean` calls the `computeInputAmount` from a primitive which calculates the `inputAmount` of `inputToken` a user must provide to get `outputAmount` of `outputToken` from the primitive

- `ComputeOutputAmount`
  - `Ocean` calls the `computeOutputAmount` from a primitive which calculates the `outputAmount` of `outputToken` that a user can get for `inputAmount` of `inputToken` provided to the primitive

- `UnwrapEther`
  - "unwraps" the Ocean wrapped Ether (shETH) by transferring the specified amount of Ether to the user from the Ocean contract
  - at the end of the interaction burns the ocean wrapped Ether (shETH) from the user


### OceanAdapter.sol

OceanAdapter is a helper abstract contract that connects the Ocean contract with other external primitives such as Curve.

It is meant to be inherited by other contracts, currently two implementations use it, namely Curve2PoolAdapter and CurveTricryptoAdapter.

It enables Ocean users to swap, provide liquidity or remove liquidity from external primitives.

It is not meant to be called by users directly, but from the central Ocean contract using interactions just like with primitives in the Shell ecosystem.

### Curve2PoolAdapter.sol

Enables Shell users to swap, provide liquidity or remove liquidity from the Curve USDC/USDT pool on Arbitrum.

Under the hood it "unwraps" the tokens the user wants to use, calls Curve2Pool to either swap, add liquidity or remove liquidity depending on the input and output tokens provided and lastly "wraps" the tokens back into the Ocean. At the end of the interaction Ocean mints the necessary ocean wrapped tokens to the user.

As stated before it uses max approve without any revoke functionality which can be dangerous in case of an exploit.

### CurveTricryptoAdapter.sol

Enables Shell users to swap, provide liquidity or remove liquidity from CurveTricrypto pool of USDT/wBTC/wETH on Arbitrum.

Under the hood it "unwraps" the tokens the user wants to use, calls CurveTricrypto to either swap, add liquidity or remove liquidity depending on the input and output tokens provided and lastly "wraps" the tokens back into the Ocean. At the end of the interaction Ocean mints the necessary ocean wrapped tokens to the user.

As stated before it uses max approve without any revoke functionality which can be dangerous in case of an exploit.

## Protocol Documentation

While Shell protocol does have decent documentation for the Ocean contract, it does not have any for other three contracts (OceanAdapter, Curve2PoolAdapter, CurveTricryptoAdapter).

## Closing thoughts

Security is a constant battle that is hard to win sometimes. I highly suggest for the developers to read an article about [How to Secure a Blockchain Application](https://hackernoon.com/how-to-secure-a-blockchain-application) and implement any advice that they see could benefit them.

As well as to be better prepared in case of a hack to read this [Crisis Handbook](https://docs.google.com/document/d/1DaAiuGFkMEMMiIuvqhePL5aDFGHJ9Ya6D04rdaldqC0/edit?pli=1#heading=h.z1laidcdabs5) to better understand what to do.

### Time spent:
40 hours