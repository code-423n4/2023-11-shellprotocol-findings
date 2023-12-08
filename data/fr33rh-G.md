
## Gas Optimizationin in the `function _erc20Wrap`

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L808-L842
```
    /**
     * @dev Wrap an ERC-20 token into the ocean. The Ocean ID is
     *  derived from the contract address and a tokenId of 0.
     * @notice Token amounts are normalized to 18 decimal places.
     * @dev This means that to wrap 5 units of token A, which has 6 decimals,
     *  and 5 units of token B, which has 18 decimals, the user would specify
     *  5 * 10**18 for both token A and B.
     * @param tokenAddress address of the ERC-20 token
     * @param amount amount of the ERC-20 token to be wrapped, in terms of
     *  18-decimal fixed point
     * @param userAddress the address of the user who is wrapping the token
     */
    function _erc20Wrap(address tokenAddress, uint256 amount, address userAddress, uint256 outputToken) private {
        try IERC20Metadata(tokenAddress).decimals() returns (uint8 decimals) {
            /// @dev the amount passed as an argument to the external token
            uint256 transferAmount;
            /// @dev the leftover amount accumulated by the Ocean.
            uint256 dust;

            (transferAmount, dust) = _determineTransferAmount(amount, decimals);

            // If the user is unwrapping a delta, the residual dust could be
            // written to the user's ledger balance. However, it costs the
            // same amount of gas to place the dust on the owner's balance,
            // and accumulation of dust may eventually result in
            // transferrable units again.
            _grantFeeToOcean(outputToken, dust);

            SafeERC20.safeTransferFrom(IERC20(tokenAddress), userAddress, address(this), transferAmount);

            emit Erc20Wrap(tokenAddress, transferAmount, amount, dust, userAddress, outputToken);
        } catch {
            revert NO_DECIMAL_METHOD();
        }
    }
```

The param `amount` is "amount of the ERC-20 token to be wrapped, in terms of
18-decimal fixed point ", 
for example, to wrap 5.12345678901234567890 units of token A, which has 20 decimals, and 5.12 units of token B, which has 2 decimals, the user would specify
`5.123456789012345678 * 10**18` for token A ,and `5.12 * 10**18` for token B.
So the value of `truncated` has been always 0 when wrap an erc20 token, resulting in the value of `dust` also being alway 0.

```
    function _determineTransferAmount(
        uint256 amount,
        uint8 decimals
    )
        private
        pure
        returns (uint256 transferAmount, uint256 dust)
    {
        // if (decimals < 18), then converting 18-decimal amount to decimals
        // transferAmount will likely result in amount being truncated. This
        // case is most likely to occur when a user is wrapping a delta as the
        // final interaction in a transaction.
        uint256 truncated;


        (transferAmount, truncated) = _convertDecimals(NORMALIZED_DECIMALS, decimals, amount);


        if (truncated > 0) {
            // Here, FLOORish(x) is not to the nearest integer less than `x`,
            // but rather to the nearest value with `decimals` precision less
            // than `x`. Likewise with CEILish(x).
            // When truncating, transferAmount is FLOORish(amount), but to
            // fully cover a potential delta, we need to transfer CEILish(amount)
            // if truncated == 0, FLOORish(amount) == CEILish(amount)
            // When truncated > 0, FLOORish(amount) + 1 == CEILish(AMOUNT)
            transferAmount += 1;
            // Now that we are transferring enough to cover the delta, we
            // need to determine how much of the token the user is actually
            // wrapping, in terms of 18-decimals.
            (uint256 normalizedTransferAmount, uint256 normalizedTruncatedAmount) =
                _convertDecimals(decimals, NORMALIZED_DECIMALS, transferAmount);
            // If we truncated earlier, converting the other direction is adding
            // precision, which cannot truncate.
            assert(normalizedTruncatedAmount == 0);
            assert(normalizedTransferAmount > amount);
            dust = normalizedTransferAmount - amount;
        } else {
            // if truncated == 0, then we don't need to do anything fancy to
            // determine transferAmount, the result _convertDecimals() returns
            // is correct.
            dust = 0;
        }
    }
```


### solution A 
Just delete this line of code in `function _erc20Wrap`.
```
            _grantFeeToOcean(outputToken, dust);
```
### solution B （recommend）
Delete the `function _determineTransferAmount`,and rewrite `function _erc20Wrap`: 
```
    function _erc20Wrap(address tokenAddress, uint256 amount, address userAddress, uint256 outputToken) private {
        try IERC20Metadata(tokenAddress).decimals() returns (uint8 decimals) {
            (uint256 transferAmount, ) =_convertDecimals(NORMALIZED_DECIMALS, decimals, amount);
            SafeERC20.safeTransferFrom(IERC20(tokenAddress), userAddress, address(this), transferAmount);
        } catch {
            revert NO_DECIMAL_METHOD();
        }
    }
```