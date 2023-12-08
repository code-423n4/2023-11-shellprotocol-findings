[L-01] 2 address tokens will not work properly on the Ocean ERC1155 Ledger.

ERC20 tokens with multiple entry points (known as double entry tokens or two address tokens) such as Synthetix's ProxyERC20 contract will not work properly on the ledger. In the ledger, all tokens are given a specific uint256 value, by calling `_getSpecifiedToken()`, which will call `_calculateOceanId()`.

```
        uint256 specifiedToken = _getSpecifiedToken(interactionType, externalContract, interaction
```

```
   function _calculateOceanId(address tokenAddress, uint256 tokenId) internal pure returns (uint256) {
        return uint256(keccak256(abi.encodePacked(tokenAddress, tokenId)));
    }
```

If a user uses a two address token, the two address will be considered different, and the Ocean will create 2 separate ledger instead of 1.

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L107

[L-02] Interactions is succeptible to read-only reentrancy

The interaction flow in Ocean.sol will complete the interaction first before minting/burning the input/output token.

```
        (inputToken, inputAmount, outputToken, outputAmount) = _executeInteraction(
                interaction, interactionType, externalContract, specifiedToken, interaction.specifiedAmount, userAddress
            );
        }

        // if _executeInteraction returned a positive value for inputAmount,
        // this amount must be deducted from the user's Ocean balance
        if (inputAmount > 0) {
            // since uint, same as (inputAmount != 0)
            _burn(userAddress, inputToken, inputAmount);
        }

        // if _executeInteraction returned a positive value for outputAmount,
        // this amount must be credited to the user's Ocean balance
        if (outputAmount > 0) {
            // since uint, same as (outputAmount != 0)
            _mint(userAddress, outputToken, outputAmount);
        }
```

Take Unwrapping ERC721 interaction as an example. The user have an shERC721 and wants to unwrap it to get his ERC721 token back. When the ERC721 token is transferred to the user, the shERC721 is not burned yet. This means that the user has both shERC721 and ERC721 in the state.

```
    function _erc721Unwrap(address tokenAddress, uint256 tokenId, address userAddress, uint256 oceanId) private {
        IERC721(tokenAddress).safeTransferFrom(address(this), userAddress, tokenId);
        emit Erc721Unwrap(tokenAddress, tokenId, userAddress, oceanId);
    }
```

If the Ocean has more interactions in the future like lending/borrowing, the user can call ERC721Unwrap and do a cross-function reentrancy by reentering `safeTransferFrom()` after getting his ERC721 back but before burning his shERC721. For example, if the lending interaction just needs user to show proof that they have a particular shERC721 on hand, user can do the following:

- WrapERC721 to get shERC721
- UnwrapERC721
- After safeTransferFrom is called on ERC721, there is a callback to onERC721Received
- User codes the ERC721Received to call the lend function
- Since the lend function only requires the existence of the shERC721, call passes and user gets his loan
- Callback ends, finishes the execution by burning the shERC721.
- User gets his ERC721 back and his loan

```
function onERC721Received(
        address _sender, address _from, uint256 _tokenId, bytes memory _data)
        external returns (bytes4 retval) {

        require(msg.sender == address(victim), "Nope");
        // Call the loan function
        return victim.onERC721Received.selector;
}
```

This is hypothetical because there is no loan/borrow function on Ocean. However, the existence of a read-only reentrancy and that the user has 2 tokens at a point in the execution is dangerous. Ideally, burn the sh(Tokens) first before interaction. Minting later is fine. At all times, the user should only have 1 token in the state.

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L904

[L-03] User can escape fees on ERC1155 tokens

ERC1155 tokens from external sources may have different decimals, but they are not normalized to 18 decimals. The Ocean ledger takes whatever decimals it is and sets it to the corresponding shToken. For example, if a user wants to wrap an ERC1155 token that has 4 decimals, then they will mint a 4 decimal shToken.

```
function _erc1155Wrap(
        address tokenAddress,
        uint256 tokenId,
        uint256 amount,
        address userAddress,
        uint256 oceanId
    )
        private
    {
        //@audit Note that there is no `_convertDecimals()` call
        if (tokenAddress == address(this)) revert NO_RECURSIVE_WRAPS();
        _ERC1155InteractionStatus = INTERACTION;
        IERC1155(tokenAddress).safeTransferFrom(userAddress, address(this), tokenId, amount, "");
        _ERC1155InteractionStatus = NOT_INTERACTION;
        emit Erc1155Wrap(tokenAddress, tokenId, amount, userAddress, oceanId);
    }
```

The fee is therefore dependent on the amount of decimals that the ERC1155 token has. If a user wraps 100 tokens with 2 decimal places, eg 100e2, they can choose not to pay the fees by calling `erc1155Unwrap` 5 times instead of unwrapping 100e2 tokens directly. This is because the unwrapFeeDivisor has a minValue of 2000, so if the user can make the unwrapAmount less than 2000, then they will not need to pay the fees. (Instead of unwrapping 10000 tokens and paying 5 tokens as fee, user unwraps 2000 tokens 5 times).

```
    function _calculateUnwrapFee(uint256 unwrapAmount) private view returns (uint256 feeCharged) {
        feeCharged = unwrapAmount / unwrapFeeDivisor;
    }
```

[L-04] Airdrops will be locked in the contract

There are some collection of ERC721 tokens that will airdrop users tokens if they have proof that they own an ERC721. There are other ERC721 collection that will airdrop tokens to every holder of the token. If the user deposits the ERC721 token which is eligible for an airdrop into the Ocean protocol, and an airdrop happens, the Ocean will receive the airdrop instead.

Since the ocean contract does not have any way to withdraw random tokens, the airdrop will be locked inside the protocol.

Not sure about the recommendation, but take note that some ERC721 collections may mint tokens for every holder or may airdrop tokens to holders once in a while. Make sure that these tokens are either not accepted into Ocean, or that the users are warned beforehand that the airdrop will be lost.

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L79

[N-01] NATSPEC on OceanAdapter.\_convertDecimals should be revised

The function `_convertDecimals()` from OceanAdapter.sol is an exact copy on Ocean.sol. However, OceanAdapter version does not return the truncated amount.

```
(OceanAdapter.sol)
function _convertDecimals(
        uint8 decimalsFrom,
        uint8 decimalsTo,
        uint256 amountToConvert
    )
        internal
        pure
>       returns (uint256 convertedAmount)
```

```
 function _convertDecimals(
        uint8 decimalsFrom,
        uint8 decimalsTo,
        uint256 amountToConvert
    )
        internal
        pure
>       returns (uint256 convertedAmount, uint256 truncatedAmount)
```

The comments on OceanAdapter.sol should be revised accordingly to not reference the truncatedAmount.

```
     * @dev convert a uint256 from one fixed point decimal basis to another,
     *   returning the truncated amount if a truncation occurs.
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L129-L130

[N-02] minimumOutputAmount in CurveAdaptor can be integrated into the curve functionalities

The Curve Adapters interacts with curve pool through three functions, `exchange()`, `remove_liquidity_one_coin()` and `add_liquidity()`. The last parameter of these functions are set to zero, which is the slippage parameter.

```
 ICurveTricrypto(primitive).remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);
```

Ocean protocol takes care of it by checking the slippage right at the end, but it can also be integrated right into the curve functions.

```
        if (uint256(minimumOutputAmount) > outputAmount) revert SLIPPAGE_LIMIT_EXCEEDED();
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L201-L203

[N-03] Comments on ERC20Wrap is misleading

In ERC20Wrap, the `amount` parameter is supposed to be the amount of shTokens to receive, not the amount of ERC20 tokens to be wrapped.

```
>    * @param amount amount of the ERC-20 token to be wrapped, in terms of
     *  18-decimal fixed point
     * @param userAddress the address of the user who is wrapping the token
     */
>   function _erc20Wrap(address tokenAddress, uint256 amount, address userAddress, uint256 outputToken) private {
```

The `outputAmount` is the `specifiedAmount`, which is passed as `amount` into ERC20Wrap function.

```
        } else if (interactionType == InteractionType.WrapErc20) {
            inputToken = 0;
            inputAmount = 0;
            outputToken = specifiedToken;
>          outputAmount = specifiedAmount;
>           _erc20Wrap(externalContract, outputAmount, userAddress, outputToken);
```

This `outputAmount` is then minted as the shToken, which means that the `outputAmount` must be in the form of shTokens.

```
        if (outputAmount > 0) {
            // since uint, same as (outputAmount != 0)
            _mint(userAddress, outputToken, outputAmount);
        }
```

It will be useful if `amount` is more specific, and better explained in the comments. Is `amount` referring to the token amount to deposit or the corresponding shToken to receive?

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L426-L431
