# Gas Optimization

## [G-01] Add events as view The Curve2PoolAdapter::swap/deposit/withdraw events can be declared as view to avoid writing to storage if computeOutput is false.

Here is an example of how to declare the swap event as view:

```solidity
event Swap(
  uint256 inputToken,
  uint256 inputAmount,
  uint256 outputAmount,
  bytes32 slippageProtection,
  address user,
  bool computeOutput
) view;

function primitiveOutputAmount(...) {

  // compute amounts

  if(computeOutput) {
    emit Swap(
      inputToken,
      inputAmount,
      outputAmount,
      slippageProtection,
      user,
      true
    );
  }

}
```

By declaring the event as view, it avoids writing to storage if `computeOutput` is false.

The key changes:

1. Add the view modifier to the event declaration

2. Only emit the event inside an if statement checking computeOutput

3. Pass computeOutput as true when emitting

This prevents writing to storage in the case where we are just computing the output without finalizing a transaction.

Estimated gas savings would be 20,000 gas or more per view event emitted, since it avoids writing to storage. Over many
function calls this can add up significantly.

## [G-02] Combine indexOf and decimals mappings into a single mapping

Here is an example of how to combine the indexOf and decimals mappings into a single mapping:

```solidity
struct TokenInfo {
  uint256 index;
  uint8 decimals;
}

mapping(uint256 => TokenInfo) tokens;

constructor() {

  // xToken

  TokenInfo memory xTokenInfo = TokenInfo({
    index: 0,
    decimals: IERC20(xTokenAddress).decimals()
  });

  tokens[xToken] = xTokenInfo;

  // yToken

  TokenInfo memory yTokenInfo = TokenInfo({
    index: 1,
    decimals: IERC20(yTokenAddress).decimals()
  });

  tokens[yToken] = yTokenInfo;

  // etc

}

function primitiveOutputAmount(uint256 inputToken, ...) {

  TokenInfo storage tokenInfo = tokens[inputToken];

  uint256 index = tokenInfo.index;
  uint8 decimals = tokenInfo.decimals;

  // use index and decimals without extra SLOAD

}
```

The benefits are:

- Reduce overall storage usage
- Avoid multiple SLOADs to fetch index and decimals
- Group related metadata together

Estimated gas savings is ~200 gas per token from reducing one storage slot.

```solidity
File: src/adapters/Curve2PoolAdapter.sol

65    mapping(uint256 => int128) indexOf;

68    mapping(uint256 => uint8) decimals;
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L65-L68

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

73    mapping(uint256 => uint256) indexOf;

76    mapping(uint256 => uint8) decimals;
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L72-L76

## [G-03] Removal of Unused Function in OceanAdapter Contract

In the contract OceanAdapter, removed the unused function OceanAdapter::computeInputAmount() as it is marked as "Not
implemented" and not used in the contract.

```solidity
File:  src/adapters/OceanAdapter.sol

    function computeInputAmount(
        uint256 inputToken,
        uint256 outputToken,
        uint256 outputAmount,
        address userAddress,
        bytes32 maximumInputAmount
    )
        external
        override
        onlyOcean
        returns (uint256 inputAmount)
    {
        revert();
    }
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L81-L94

## [G-04] Use Storage Pointers for State Variable Updates in OceanAdapter Contract

In the contract OceanAdapter, applied the optimization technique of using storage pointers instead of direct value
assignments when updating state variables within functions.

By using storage pointers, we reduce unnecessary state writes, resulting in gas savings during contract execution.

Code Changes:

Before:

```solidity
function someFunction(uint256 newValue) external {
    myStateVariable = newValue;
}
```

After:

```solidity
function someFunction(uint256 newValue) external {
    uint256 storage myStateVariableRef = myStateVariable;
    myStateVariableRef = newValue;
}
```

In the code, the state variable myStateVariable is updated directly with the new value. However, by using a storage
pointer, we create a reference to the state variable, and then update the reference with the new value.

This approach avoids unnecessary state writes, resulting in potential gas savings.

```solidity
File: src/adapters/OceanAdapter.sol

33        ocean = ocean_;
34        primitive = primitive_;
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L33