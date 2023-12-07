
# [QA-01] Incorrect comment in `CurveTricryptoAdapter.sol` for `_determineComputeType()`

[File: CurveTricryptoAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L257-L263)
```
    /**
     * @dev Uses the inputToken and outputToken to determine the ComputeType
     *  (input: xToken, output: yToken) | (input: yToken, output: xToken) => SWAP
     *  base := xToken | yToken
     *  (input: base, output: lpToken) => DEPOSIT
     *  (input: lpToken, output: base) => WITHDRAW
     */
```

`CurveTricryptoAdapter.sol` uses `xToken`, `yToken` and `zToken`. The above comment misses `zToken`. It looks like it was copy-pasted from `Curve2PoolAdapter.sol`, where, indeed, are only `xToken` and `zToken`.

The correct version of the comment should be:

```
    /**
     * @dev Uses the inputToken and outputToken to determine the ComputeType
     *  (input: xToken, output: yToken) | (input: yToken, output: xToken) 
     *  | (input: xToken, output: zToken) | (input: zToken, output: xToken) 
     *  | (input: yToken, output: zToken) | (input: zToken, output: yToken)=> SWAP
     *  base := xToken | yToken
     *  (input: base, output: lpToken) => DEPOSIT
     *  (input: lpToken, output: base) => WITHDRAW
     */
```

# [QA-02] `changeUnwrapFee()` does not verify if new value of unwrap fee was provided.

While changing unwrap fee, function `changeUnwrapFee()` does not verify if we indeed, provided new value.
It's a good practice to check if value was indeed changed, before emitting an event.

[File: Ocean.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L196)
```
    function changeUnwrapFee(uint256 nextUnwrapFeeDivisor) external override onlyOwner {
        /// @notice as the divisor gets smaller, the fee charged gets larger
        if (MIN_UNWRAP_FEE_DIVISOR > nextUnwrapFeeDivisor) revert();
        emit ChangeUnwrapFee(unwrapFeeDivisor, nextUnwrapFeeDivisor, msg.sender);
        unwrapFeeDivisor = nextUnwrapFeeDivisor;
    }
```

E.g. if the current value of `unwrapFeeDivisor` is `5000` and we will call `changeUnwrapFee(5000)`, function will emit an event (`ChangeUnwrapFee()`), even though, nothing will be changed (`unwrapFeeDivisor` will still be `5000`). It's a good practice to verify if the provided value is different than the one we're changing. Our recommendation is to add below, additional check:

```
require(nextUnwrapFeeDivisor != unwrapFeeDivisor, "Nothing is being changed!");
```


# [QA-02] `onERC1155BatchReceived` in `Ocean.sol` is not compliant with EIP-1155

Even though the Ocean never initiates ERC1155 Batch Transfers, while devs decided to implement `onERC1155BatchReceived` - it must be compliant with EIP-1155.

According to EIP-1155:

```
https://eips.ethereum.org/EIPS/eip-1155

onERC1155BatchReceived rules:
[...]
If the return value is bytes4(keccak256("onERC1155Received(address,address,uint256,uint256,bytes)")) the transfer MUST be completed or MUST revert if any other conditions are not met for success.
[....]
If the return value is anything other than bytes4(keccak256("onERC1155Received(address,address,uint256,uint256,bytes)")) the transaction MUST be reverted.
```

The current implementation of `onERC1155BatchReceived()` in `Ocean.sol` always returns 0 which means that transfers will always revert. 
Make sure, that `onERC1155BatchReceived()` returns `IERC1155Receiver.onERC1155BatchReceived.selector` instead of 0. If it returns 0, every transfer which wants to be compliant with EIP-1155 will always revert.


# [QA-03] Slippage limit represented as bytes32 may be misleading to the end-user

The slippage protection is represented as `bytes32` and then casted to `uint256`. This behavior may be misleading to the end user who expects to provide the slippage limit as an amount.
Slippage protects from getting smaller outputAmount than expected. Since `outputAmount` is `uint256`, then slippage protection should also be `uint256`.

[File: Curve2PoolAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L142)
```
function primitiveOutputAmount(
        uint256 inputToken,
        uint256 outputToken,
        uint256 inputAmount,
        bytes32 minimumOutputAmount
    )

    [...]

    if (uint256(minimumOutputAmount) > outputAmount) revert SLIPPAGE_LIMIT_EXCEEDED();

    [...]

     if (action == ComputeType.Swap) {
            emit Swap(inputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);
        } else if (action == ComputeType.Deposit) {
            emit Deposit(inputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);
        } else {
            emit Withdraw(outputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);
        }
```
While `uint256` can be represented in decimals, `bytes32` will be always in hex. The `bytes32` will be both provided as a paremeter and emitted in an event. Having that kind of representation (treating slippage as `bytes32` instead of `uint256`) may be misleading to the user.

The same issue occurs in `CurveTricryptoAdapter.sol` in `primitiveOutputAmount` function.




# [NC-01] Grammar errors


[File: CurveTricryptoAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L172)
```
 * @dev swaps/add liquidity/remove liquidity from Curve Tricrypto Pool
```

`swaps/add liquidity/remove` should be changed to `swaps/adds liquidity/removes`.

[File: OceanAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L45)
```
do the accounting.  One of the token amounts is chosen by the user, and
```

`token amounts` should be changed to `token's amount`.

[File: Curve2PoolAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L136)
```
* @dev swaps/add liquidity/remove liquidity from Curve 2pool
```
`swaps/add liquidity/remove` should be changed to `swaps/adds liquidity/removes`.



# [NC-02] Do not revert with empty messages

Make sure, that `revert()` provides a message string which explains why the revert occurred.

[File: OceanAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L93)
```
revert();
```

[File: Ocean.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L198)
```
if (MIN_UNWRAP_FEE_DIVISOR > nextUnwrapFeeDivisor) revert();
```


# [NC-03] Provide more descriptive explanation of packed values

[File: OceanAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L99)
```
    function _fetchInteractionId(address token, uint256 interactionType) internal pure returns (bytes32) {
        uint256 packedValue = uint256(uint160(token));
        packedValue |= interactionType << 248;
        return bytes32(abi.encode(packedValue));
    }
```

Function `_fetchInteractionId()` packs `address token` and `uint256 interactionType` into `bytes32`. Usually, packing variables may be confusing for the reader. It's a good coding-practice to visualize how the variable will be packed, so the end-user will be easily able to know at which byte, each variable lies.


# [N-04] Stick with one grammar tense when describing functions in the NatSpace

Code-base uses different tenses when describing the purpose of the function. This issue occurs in the whole code-base. To better visualize the problem, we'll give an example from `OceanAdapter`

```
 	/**
     * @notice calculates Ocean ID for a underlying token
     */
    function _calculateOceanId(address tokenAddress, uint256 tokenId) internal pure returns (uint256) {

   [...]

    /**
     * @notice returning 0 here since this primitive should not have any tokens
     */
    function getTokenSupply(uint256 tokenId) external view override returns (uint256) {

    [...]

    /**
     * @notice used to fetch the Ocean interaction ID
     */
    function _fetchInteractionId(address token, uint256 interactionType) internal pure returns (bytes32) {
```

For better readibility, stick with the one grammar-tense in comments. In above example `_calculateOceanId()` uses Present Simple (`calculates`), `getTokenSupply()` uses Present Continous (`returning`), `_fetchInteractionId()` uses Past Simple (`used`).


# [N-05] Errors should not be UPPER-cased.

Stick with coding convention by naming errors like: `error Error()`, instead of `error ERROR()`. All-uppercase letters should be reserved for constants only.

[File: CurveTricryptoAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L29-L30)
```
    error INVALID_COMPUTE_TYPE();
    error SLIPPAGE_LIMIT_EXCEEDED();
```


[File: Curve2PoolAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L24-L25)
```
    error INVALID_COMPUTE_TYPE();
    error SLIPPAGE_LIMIT_EXCEEDED();
```


# [N-06] Use descriptive constant values, instead of numbers

The code readability will be increased, when numbers 0, 1, 2 will be assigned to constants with descriptive names:

[File: CurveTricryptoAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L86)
```
		address xTokenAddress = ICurveTricrypto(primitive).coins(0);
        xToken = _calculateOceanId(xTokenAddress, 0);

        [...]

        address yTokenAddress = ICurveTricrypto(primitive).coins(1);
        yToken = _calculateOceanId(yTokenAddress, 0);

        [...]

        address wethAddress = ICurveTricrypto(primitive).coins(2);

        [...]

        indexOf[zToken] = 2;
```

The same issue occurs in `Curve2PoolAdapter.sol` `constructor()`.


# [N-07] Address of `zToken` can be pre-calculated

[File: CurveTricryptoAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L100)
```
zToken = _calculateOceanId(address(0x4574686572), 0); // hexadecimal(ascii("Ether"))
```

Since `_calculateOceanId()` returns `uint256(keccak256(abi.encodePacked(tokenAddress, tokenId)))` and we provide it with already known values: `address(0x4574686572)` and `0`, the `zToken` address can be pre-calculated.



# [N-08] `primitiveOutputAmount()` - emit an event after the action

In `CurveTricryptoAdapter.sol`, there are two `if`/`else` conditional checks. The first check verifies the `action` (if `action` is `Swap`, `Deposit`, `Withdraw`) and performs the appropriate operations. Then, another check verifies the same (if `action` is `Swap`, `Deposit`, `Withdraw`) and then emits the appropriate event. Using the second `if`/`else` conditional check is, however, redundant. The appropriate event should be emitted just after the action.

[File: CurveTricryptoAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L229)
```
 if (action == ComputeType.Swap) {
            emit Swap(inputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);
        } else if (action == ComputeType.Deposit) {
            emit Deposit(inputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);
        } else {
            emit Withdraw(outputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);
        }
```

Instead of performing the same `if`/`else` check, emit events just after appropriate actions:

```
 if (action == ComputeType.Swap) {
            bool useEth = inputToken == zToken || outputToken == zToken;

            ICurveTricrypto(primitive).exchange{ value: inputToken == zToken ? rawInputAmount : 0 }(
                indexOfInputAmount, indexOfOutputAmount, rawInputAmount, 0, useEth
            );
        uint256 rawOutputAmount = _getBalance(underlying[outputToken]) - _balanceBefore;
        outputAmount = _convertDecimals(decimals[outputToken], NORMALIZED_DECIMALS, rawOutputAmount);

        emit Swap(inputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);

        } else if (action == ComputeType.Deposit) {

       		uint256[3] memory inputAmounts;
            inputAmounts[indexOfInputAmount] = rawInputAmount;

            if (inputToken == zToken) IWETH(underlying[zToken]).deposit{ value: rawInputAmount }();

            ICurveTricrypto(primitive).add_liquidity(inputAmounts, 0);

            uint256 rawOutputAmount = _getBalance(underlying[outputToken]) - _balanceBefore;
        	outputAmount = _convertDecimals(decimals[outputToken], NORMALIZED_DECIMALS, rawOutputAmount);

        	 emit Deposit(inputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);

        }

        [...]
```

The very same issue occurs in `Curve2PoolAdapter.sol` in `primitiveOutputAmount()`.


# [N-09] Stick with one way of representing `if`/`else` conditions

Whenever there's only one operation in `if`/`else` condition, brackets can be omitted, e.g.: `if(condition) {action;}` can be rewritten to `if(condition) action;`.
However, in the whole code-base, developers used the first approach (with the brackets):

[File: CurveTricryptoAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L229)
```
if (action == ComputeType.Swap) {
            emit Swap(inputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);
        } else if (action == ComputeType.Deposit) {
            emit Deposit(inputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);
        } else {
            emit Withdraw(outputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);
        }
```

As demonstrated above, even when there's only one instruction, brackets are being used (even though they can be omitted).

The only exception from this rule has been observed in `CurveTricryptoAdapter.sol` at line 208:

[File: CurveTricryptoAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L208)
```
if (inputToken == zToken) IWETH(underlying[zToken]).deposit{ value: rawInputAmount }();
```

To stick to one convention, it's recommended to change above code to:
```
if (inputToken == zToken) {
	IWETH(underlying[zToken]).deposit{ value: rawInputAmount }();
}
```


# [N-10] Define constants for `type(uint256).max`
Defining constants for `type(uint256).max` would allow to better explain the purpose of the `type(uint256).max`

[File: CurveTricryptoAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L242-L243)
```
        IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);
        IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);
```


[File: Curve2PoolAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L190-L191)
```
        IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);
        IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);
```


# [N-11] Typos

* `Ocean.sol`

```
30:		* @title A public multitoken ledger for defi // defi -> DeFi
35:		 1. At the highest level, it is a defi framework[0]. Users provide a list // defi -> DeFi
38: 	 *  2. Suporting this defi framework is an accounting system that can transfer, // defi -> DeFi
65:		 *  3. Read through the imlementation of Ocean.doMultipleInteractions(), again  // imlementation -> implementation
165: 	 * @notice initializes the fee divisor to uint256 max, which results in  // uint256 max -> type(uint256).max

```


# [N-12] Use `uint256` instead of `uint`
Even though `uint` is shorter version of `uint256`, using `uint256` across the whole code-base (including comments), increases the code readibility.

[File: Ocean.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L420-L427)
```
if (inputAmount > 0) {
            // since uint, same as (inputAmount != 0)
[...]
            // since uint, same as (outputAmount != 0)
```