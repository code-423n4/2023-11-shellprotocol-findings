# Low - Precision Loss
Inside `_convertDecimals()` if `amountToConvert < decimalsFrom - decimalsTo`, this rounds to 0. Decimals between 0 and 255 are valid values. This can cause needless reverts upstream for users if the primitive doesn't properly adjust the users specified amounts to the ocean standard of 1e18. An inputAmount of 0 causes the outputAmount to be 0.

Moreover, if the `minimumOutputAmount` parameter in `primitiveOutputAmount()` in the adapter files is set to bytes32(0), then the check `if (uint256(minimumOutputAmount) > outputAmount) revert SLIPPAGE_LIMIT_EXCEEDED();` will pass and emit a swap/deposit/withdraw event with 0 values, but nothing will end up being burned or minted in `doInteractions()` and `doMultipleInteractions()`.

Example Calculation: Alice has 500 USDC (6 decimals), and the primitive does not properly convert their inputAmount from 500e6 to 500e18. Their input amount gets rounded to zero through the following calculations:


```
// rawInputAmount calculation:
uint256 rawInputAmount = _convertDecimals(NORMALIZED_DECIMALS, decimals[inputToken], inputAmount);

// Relevant _convertDecimals function():
uint256 shift = 10 ** (uint256(decimalsFrom - decimalsTo));
convertedAmount = amountToConvert / shift;
```

```
uint256 rawInputAmount = _converDecimals(18, 6, 500e6)
uint256 shift = 10 ** (18 - 6) = 1e12
convertedAmount = 500e6 / 1e12 = 0
```

Possible remedy with check and custom error:


```diff
// Ocean.sol
+   error AMOUNT_TOO_SMALL();

    function _convertDecimals(
        uint8 decimalsFrom,
        uint8 decimalsTo,
        uint256 amountToConvert
    )
        internal
        pure
        returns (uint256 convertedAmount, uint256 truncatedAmount)
    {

        if (decimalsFrom == decimalsTo) {
            // no shift
            convertedAmount = amountToConvert;
            truncatedAmount = 0;
        } else if (decimalsFrom < decimalsTo) {
            // Decimal shift left (add precision)
            uint256 shift = 10 ** (uint256(decimalsTo - decimalsFrom));
            convertedAmount = amountToConvert * shift;
            truncatedAmount = 0;
        } else {
            // Decimal shift right (remove precision) -> truncation
            uint256 shift = 10 ** (uint256(decimalsFrom - decimalsTo));
+           if (amountToConvert < shift) revert AMOUNT_TOO_SMALL();
            convertedAmount = amountToConvert / shift;
            truncatedAmount = amountToConvert % shift;
        }
    }
```

```diff
// OceanAdapter.sol
+   error AMOUNT_TOO_SMALL();

    function _convertDecimals(
        uint8 decimalsFrom,
        uint8 decimalsTo,
        uint256 amountToConvert
    )
        internal
        pure
        returns (uint256 convertedAmount)
    {

        if (decimalsFrom == decimalsTo) {
            // no shift
            convertedAmount = amountToConvert;
        } else if (decimalsFrom < decimalsTo) {
            // Decimal shift left (add precision)
            uint256 shift = 10 ** (uint256(decimalsTo - decimalsFrom));
            convertedAmount = amountToConvert * shift;
        } else {
            // Decimal shift right (remove precision) -> truncation
            uint256 shift = 10 ** (uint256(decimalsFrom - decimalsTo));
+           if (amountToConvert < shift) revert AMOUNT_TOO_SMALL();
            convertedAmount = amountToConvert / shift;
        }
    }
```

* CurveTricryptoAdapter.sol primitiveOutputAmount(): https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L178-L236
* Curve2PoolAdapter.sol primitiveOutputAmount(): https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L142-L184
* rawInputAmount calculation in 2pool: https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L152
* rawInputAmount calculation in tricrypto: https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L188
* _convertDecimals() in Ocean: https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L1143-#L1144
* _convertDecimals() in OceanAdapter: https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L156-L157


# Low - 100% slippage for users in Curve Adapters if Primitives do not adjust metadata from default
Inside the `primitiveOutputAmount()` functions for `Curve2PoolAdapter.sol` and `CurveTricryptoAdapter.sol`, all slippage parameters for function calls to the Curve protocol are set to 0 representing 100% slippage. The function calls are `add_liquidity()`, `exchange()`, and `remove_liquidity_one_coin()`. The corresponding 0 values inside the curve contract are `min_mint_amount`, `_min_dy`, and  `min_received`. They all check if the resulting values are >= the minimums. Setting these minimums to 0 will pass the checks and allow for 100% slippage for users for tokens minted, swapped, or received.

The security is in the hands of the primitive, if they accidentally set the `interactions.metadata` to 0, which is the default of a bytes32, which is the `minimumOutputAmount`, then there is 100% slippage because this check will pass `if (uint256(minimumOutputAmount) > outputAmount) revert SLIPPAGE_LIMIT_EXCEEDED();` in the `primitiveOutputAmount()` function.

* https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L178-L236
* https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L142-L184
* https://arbiscan.io/address/0x7f90122BF0700F9E7e1F688fe926940E8839F353#code#L370
* https://arbiscan.io/address/0x7f90122BF0700F9E7e1F688fe926940E8839F353#code#L523
* https://arbiscan.io/address/0x7f90122BF0700F9E7e1F688fe926940E8839F353#code#L760


# Informational - Function Rename

Consider changing the name of `_fetchInteractionId()` to `_packInteractionTypeAndAddress()` to avoid confusion and provide clarity. `_fetchInteractionId()` is supposed to be the exact antithesis to the `_unpackInteractionTypeAndAddress()` function found in `Ocean.sol`. It takes in an address and uint256 and packs them together nicely into a bytes32 type, to be unpacked with `_unpackInteractionTypeAndAddress()` and used. There is no "Interaction ID" documented, and the function is supposed to simply pack an address and number into a bytes32 return.

Locations of `_fetchInteractionId()` in scope:
* https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L99
* Used across tests and mocks as well: MockPrimitive.sol, RecursiveMaliciousPrimitive.sol, TestCurve2PoolAdapter.t.sol, TestCurveTricryptoAdapter.t.sol


# Informational - Natspec param typo
Change `* @param WrapErc1155` to `* @param UnwrapErc1155` on line 71 in `Interactions.sol`. The `WrapErc1155` param is typed twice, so the second one should become `UnwrapErc1155`.

* https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Interactions.sol#L71

# Informational - Incorrect code in whitepaper

The Interaction struct on the whitepaper page 33 has a typo. It is supposed to show a swap from shUSDT to shDAI, but it is showing a swap from shUSDC to shUSDT.
```diff
Interaction swapUSDC {
    bytes32 interactionTypeAndAddress = {ComputeOutputAmount, AMM2};
+   uint256 inputToken = shUSDT
+   uint256 outputToken = shDAI;
-   uint256 inputToken = shUSDC;
-   uint256 outputToken = shUSDT;
    uint256 specifiedAmount = max;
    bytes32 metadata = 0;
}
```