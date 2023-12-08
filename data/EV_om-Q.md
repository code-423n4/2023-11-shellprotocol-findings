# Shell Protocol QA Report


## [L-01] Potential Overcharge on Unwrapping Fee

### Context

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L70-L74

### Description

In the `computeOutputAmount` function of `OceanAdapter.sol`, the `unwrappedAmount` is calculated by subtracting the `unwrapFee` from the `inputAmount`. This `unwrappedAmount` is then passed as a parameter to the `primitiveOutputAmount` function. However, the unwrap fee may be higher than the `unwrappedAmount` due to truncation, in which case the value passed to the `primitiveOutputAmount` function is higher than the actual available amount. 

The code in question:

``
uint256 unwrapFee = inputAmount / IOceanInteractions(ocean).unwrapFeeDivisor();
uint256 unwrappedAmount = inputAmount - unwrapFee;
outputAmount = primitiveOutputAmount(inputToken, outputToken, unwrappedAmount, metadata);
``

The code implicitly assumes that Adapters need to truncate the `unwrappedAmount` to the token's decimals. 

### Recommendation 

Instead of making the assumption about the truncation, the unwrapped amount should be inferred from the balance change and passed to `primitiveOutputAmount`, ideally already in the token's precision. This would ensure that the correct amount is used in the calculations and prevent potential overcharging.


## [L-02] Incorrect Event Parameter in Swap Action

### Context

`Curve2PoolAdapter.sol#L100`

### Description

In the `primitiveOutputAmount` function of the `Curve2PoolAdapter` contract, the `inputAmount` parameter is passed to the `Swap` event without being converted to `rawInputAmount`. This could potentially lead to incorrect event logs as the `inputAmount` might not reflect the actual amount used in the swap operation due to decimal conversions. The `rawInputAmount` is the value that is actually used in the swap operation, and it is derived from `inputAmount` after adjusting for decimal differences.

Relevant code:
```
if (action == ComputeType.Swap) {
    emit Swap(inputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);
}
```

### Recommendation 

To ensure the event logs accurately reflect the operations performed, it is recommended to emit the `Swap` event with `rawInputAmount` instead of `inputAmount`. This will ensure that the logged amount is the actual amount used in the swap operation. The code should be updated as follows:

```
if (action == ComputeType.Swap) {
    emit Swap(inputToken, rawInputAmount, outputAmount, minimumOutputAmount, primitive, true);
}
```


## [L-03] Incorrect Documentation Regarding int256 Maximum Value

### Context

`src/ocean/BalanceDelta.sol#L100`

### Description

The comment in the code suggests that the maximum value of `int256` is one unit higher than the absolute value of the minimum value. However, this is incorrect. The maximum value of `int256` is actually one unit lower than the absolute value of the minimum value. This discrepancy could lead to confusion and potential errors in the future if the code is misunderstood. The relevant code is:

```
if (uint256(type(int256).max) <= amount) revert CAST_AMOUNT_EXCEEDED();
```

In addition, given the above, it is also safe to use strict equality in the above comparison, since type(int256).max is safely castable to int256 (and also has a negative representation).

### Recommendation 

The documentation should be corrected to accurately reflect the properties of `int256`. The correct statement should be: "the maximum value of `int256` is one unit lower than the absolute value of the minimum value". This will ensure that the documentation is accurate and does not lead to potential misunderstandings or errors in the future.


## [L-04] Unnecessary Fallback Function

### Context


https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L30-L53

### Description

The contract `CurveTricryptoAdapter` includes a fallback function that is not necessary. The fallback function is an unnamed function that is executed whenever the contract is called with function data that does not match any of the existing functions. However, in this case, it is not required and can be replaced with a `receive()` function which is triggered when the call data is empty.

### Recommendation 

Remove the fallback function and replace it with a `receive()` function. The `receive()` function is a special function that has no arguments, cannot return anything and must have external visibility. It is executed on a call to the contract with empty calldata.
```
receive() external payable { }
```


## [L-05] Missing Sender Address Check

### Context

`CurveTricryptoAdapter.sol#L100`

### Description

The fallback function in the `CurveTricryptoAdapter` contract does not include a check for the sender's address. This could potentially lead to Ether being locked in the contract if it is sent from an address that is not expected or allowed. 

### Recommendation 

Add a check for the sender's address in the fallback function to ensure that Ether is only accepted from expected or allowed addresses. This can be done using a modifier or a simple `require()` statement. 
```
fallback() external payable { 
    require(msg.sender == expectedAddress, "Unexpected sender");
}
```
Please replace `expectedAddress` with the actual address or condition that needs to be checked.


## [N-01] slippageProtection type and naming

### Context

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L30-L53

### Description

In the `Swap`, `Deposit` and `Withdraw` event declaration, the `slippageProtection` parameter is currently of type `bytes32`. However, considering its usage and context, it would make more sense for it to be of type `uint256`. 

Additionally, the name `slippageProtection` could be improved to better reflect its purpose. A more appropriate name could be `minOutput`, which clearly indicates that it represents the minimum output amount the user expects to receive, providing protection against price slippage.

```
event Swap(
    uint256 inputToken,
    uint256 inputAmount,
    uint256 outputAmount,
    bytes32 slippageProtection,
    address user,
    bool computeOutput
);
```

### Recommendation 

Change the type of `slippageProtection` to `uint256` and rename it to `minOutput`.


## [N-02] Inconsistent Accounting of ETH Wrapping as Interactions

### Context

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol

### Description

In the Ocean contract, there is an inconsistency in how the wrapping of ETH is accounted for as interactions. When ETH is wrapped in a call to `doInteraction` or `forwardedDoInteraction`, this is counted as an interaction in the `numberOfInteractions` parameter of the emitted `OceanTransaction` event. However, if ETH is wrapped as part of a call to `doMultipleInteractions` or `forwardedDoMultipleInteractions`, the emitted `OceanTransaction` event does not account for that action as an interaction. This inconsistency could lead to confusion and potential misinterpretation of the events.

### Recommendation 

It is recommended to check for `msg.value` in the `doMultipleInteractions` and `forwardedDoMultipleInteractions` functions and add 1 to the number of interactions if present. This would ensure that the wrapping of ETH is consistently accounted for as an interaction across all relevant functions.


## [N-03] Potential Revert in _getNegativeBalanceDelta Function

### Context

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/BalanceDelta.sol


### Description

In the `_getNegativeBalanceDelta` function, there is a potential edge case where the function will revert if `amount` equals `type(int256).min`. This is due to the line of code `return uint256(-amount);`. In the case where `amount` equals `type(int256).min`, negating `amount` will result in a value that cannot be represented as an `int256`, causing a revert.

```
function _getNegativeBalanceDelta(BalanceDelta[] memory self, uint256 tokenId) private pure returns (uint256) {
    uint256 index = _findIndexOfTokenId(self, tokenId);
    int256 amount = self[index].delta;
    if (amount > 0) revert DELTA_AMOUNT_IS_POSITIVE();
    return uint256(-amount);
}
```

While this is an edge case, it could potentially disrupt the execution of transactions that hit this condition.

### Recommendation 

To handle this edge case, consider adding a specific condition to check if `amount` equals `type(int256).min` before the negation operation. If it does, handle it appropriately based on the business logic of your contract. This could involve reverting with a more descriptive error message, or assigning a maximum possible value to the negated `amount`.


## [N-04] Misleading Function Name in Curve2PoolAdapter Contracts

### Context

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol

### Description

In the `Curve2PoolAdapter` and `CurveTricryptoAdapter` contracts, the function `_determineComputeType` not only determines the type of computation (Swap, Deposit, Withdraw) based on the input and output tokens, but also validates the tokens. The function name `_determineComputeType` does not accurately reflect this validation behavior. Misleading function names can lead to confusion for developers and auditors, potentially leading to overlooked issues or bugs.

Relevant code:
```
if (((inputToken == xToken) && (outputToken == yToken)) || ((inputToken == yToken) && (outputToken == xToken)))
```

### Recommendation 

To improve code readability and maintainability, consider renaming the function to `_determineComputeTypeAndValidateTokens`. This name more accurately describes the function's behavior, which includes both determining the computation type and validating the tokens.


## [N-05] Misleading Variable Name

### Context

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol

### Description

In the `primitiveOutputAmount` function, the variable `indexOfInputAmount` is used to store the index of the input token in the Curve pool. The current name of the variable suggests that it is storing the index of an amount, which is misleading. This could lead to confusion for developers reading or maintaining the code. The relevant code is:

```
int128 indexOfInputAmount = indexOf[inputToken];
```

### Recommendation 

To improve code readability and maintainability, it is recommended to rename the variable `indexOfInputAmount` to `indexOfInputToken`. This name accurately reflects the data that the variable is storing.


## [N-01] Inconsistent Variable Naming for IDs

### Context

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol

### Description

In the `Curve2PoolAdapter` contract, there is an inconsistency in the naming of variables representing Ocean IDs. The variables `xToken` and `yToken` are named without the 'Id' suffix, while `lpTokenId` includes the 'Id' suffix. This inconsistency can lead to confusion and potential errors when reading or modifying the code. 

Relevant code:
```
    /// @notice x token Ocean ID.
    uint256 public immutable xToken;

    /// @notice y token Ocean ID.
    uint256 public immutable yToken;

    /// @notice lp token Ocean ID.
    uint256 public immutable lpTokenId;
```

### Recommendation 

To improve code readability and maintainability, it is recommended to follow a consistent naming convention for similar variables. In this case, either add 'Id' to `xToken` and `yToken` to become `xTokenId` and `yTokenId`, or remove 'Id' from `lpTokenId` to become `lpToken`.

