# Gas Report

## Summary

Total **24 instances** over **8 issues**with **54921** gas saved:

|ID|Issue|Instances|Gas|
|:--:|:---|:--:|:--:|
| [[G-01]](#g-01-using-private-for-constants-saves-gas) | Using `private` for constants saves gas | 10 | 34060 |
| [[G-02]](#g-02-use-unchecked-block-for-safe-subtractions) | Use `unchecked` block for safe subtractions | 5 | 425 |
| [[G-03]](#g-03-do-not-cache-state-variables-that-are-used-only-once) | Do not cache state variables that are used only once | 2 | 6 |
| [[G-04]](#g-04-private-functions-only-used-once-can-be-inlined-to-save-gas) | Private functions only used once can be inlined to save gas | 1 | 30 |
| [[G-05]](#g-05-use-assembly-to-validate-msgsender) | Use assembly to validate `msg.sender` | 1 | 12 |
| [[G-06]](#g-06-state-variables-that-are-used-multiple-times-in-a-function-should-be-cached-in-stack-variables) | State variables that are used multiple times in a function should be cached in stack variables | 2 | 388 |
| [[G-07]](#g-07-contracts-can-use-fewer-storage-slots-by-truncating-state-variables) | Contracts can use fewer storage slots by truncating state variables | 1 | 20000 |
| [[G-08]](#g-08-using-constants-instead-of-enum-can-save-gas) | Using `constant`s instead of `enum` can save gas | 2 | - |

## Gas Optimizations

### [G-01] Using `private` for constants saves gas

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

<details>
<summary>There are 10 instances (click to show):</summary>

- *Curve2PoolAdapter.sol* ( [56](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L56), [59](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L59), [62](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L62) ):

```solidity
56:     uint256 public immutable xToken;

59:     uint256 public immutable yToken;

62:     uint256 public immutable lpTokenId;
```

- *CurveTricryptoAdapter.sol* ( [61](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L61), [64](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L64), [67](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L67), [70](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L70) ):

```solidity
61:     uint256 public immutable xToken;

64:     uint256 public immutable yToken;

67:     uint256 public immutable zToken;

70:     uint256 public immutable lpTokenId;
```

- *OceanAdapter.sol* ( [19](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L19), [22](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L22) ):

```solidity
19:     address public immutable ocean;

22:     address public immutable primitive;
```

- *Ocean.sol* ( [84](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L84) ):

```solidity
84:     uint256 public immutable WRAPPED_ETHER_ID;
```

</details>

### [G-02] Use `unchecked` block for safe subtractions

If it can be confirmed that the subtraction operation will not overflow, using an unchecked block can save gas.
For example, `require(x <= y); z = y - x;` can be optimized to `require(x <= y); unchecked { z = y - x; }`.

There are 5 instances:

- *OceanAdapter.sol* ( [152](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L152), [156](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L156) ):

```solidity
/// @audit checked on line 150
152:             uint256 shift = 10 ** (uint256(decimalsTo - decimalsFrom));

/// @audit checked on line 150
156:             uint256 shift = 10 ** (uint256(decimalsFrom - decimalsTo));
```

- *Ocean.sol* ( [1102](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L1102), [1138](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L1138), [1143](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L1143) ):

```solidity
/// @audit checked on line 1101
1102:             dust = normalizedTransferAmount - amount;

/// @audit checked on line 1136
1138:             uint256 shift = 10 ** (uint256(decimalsTo - decimalsFrom));

/// @audit checked on line 1136
1143:             uint256 shift = 10 ** (uint256(decimalsFrom - decimalsTo));
```

### [G-03] Do not cache state variables that are used only once

It's cheaper to access the state variable directly if it is accessed only once. This can save some gas cost of the extra stack allocation.

There are 2 instances:

- *Curve2PoolAdapter.sol* ( [103](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L103), [122](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L122) ):

```solidity
103:         address tokenAddress = underlying[tokenId];

122:         address tokenAddress = underlying[tokenId];
```

### [G-04] Private functions only used once can be inlined to save gas

If a `private` function is only used once, there is no need to modularize it, unless the function calling it would otherwise be too long and complex.

There is 1 instance:

- *Ocean.sol* ( [903](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L903) ):

```solidity
903:     function _erc721Unwrap(address tokenAddress, uint256 tokenId, address userAddress, uint256 oceanId) private {
```

### [G-05] Use assembly to validate `msg.sender`

We can use assembly to efficiently validate msg.sender with the least amount of opcodes necessary. For more details check the following report [Here](https://code4rena.com/reports/2023-05-juicebox#g-06-use-assembly-to-validate-msgsender)

There is 1 instance:

- *OceanAdapter.sol* ( [39](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L39) ):

```solidity
39:         require(msg.sender == ocean);
```

### [G-06] State variables that are used multiple times in a function should be cached in stack variables

When performing multiple operations on a state variable in a function, it is recommended to cache it first. Either multiple reads or multiple writes to a state variable can save gas by caching it on the stack.
Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.
*Saves 100 gas per instance*.

There are 2 instances:

- *Ocean.sol* ( [889](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L889), [920](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L920) ):

```solidity
/// @audit _ERC721InteractionStatus: 2 writes
889:     function _erc721Wrap(address tokenAddress, uint256 tokenId, address userAddress, uint256 oceanId) private {

/// @audit _ERC1155InteractionStatus: 2 writes
920:     function _erc1155Wrap(
```

### [G-07] Contracts can use fewer storage slots by truncating state variables

Some state variables can be safely modified so that the contract uses fewer storage slots. Each saved slot can avoid an extra Gsset (**20000 gas**) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings.

There is 1 instance:

- *Ocean.sol* ( [79](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L79) ):

```solidity
/// @audit Truncate `_ERC1155InteractionStatus` to `uint8`
/// @audit Truncate `_ERC721InteractionStatus` to `uint8`
/// @audit Fewer storage usage (3 slots -> 2 slots):
///        32 B: uint256 unwrapFeeDivisor
///        4 B: uint8 _ERC1155InteractionStatus
///        4 B: uint8 _ERC721InteractionStatus
79: contract Ocean is IOceanInteractions, IOceanFeeChange, OceanERC1155, IERC721Receiver, IERC1155Receiver {
```

### [G-08] Using `constant`s instead of `enum` can save gas

`Enum` is expensive and it is [more efficient to use constants](https://www.codehawks.com/finding/clm84992q02j9w9ruebun36d9) instead. An illustrative example of this approach can be found in the [ReentrancyGuard.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/181d518609a9f006fcb97af63e6952e603cf100e/contracts/utils/ReentrancyGuard.sol#L34-L35).

There are 2 instances:

- *Curve2PoolAdapter.sol* ( [10-14](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L10-L14) ):

```solidity
10: enum ComputeType {
11:     Deposit,
12:     Swap,
13:     Withdraw
14: }
```

- *CurveTricryptoAdapter.sol* ( [15-19](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L15-L19) ):

```solidity
15: enum ComputeType {
16:     Deposit,
17:     Swap,
18:     Withdraw
19: }
```

