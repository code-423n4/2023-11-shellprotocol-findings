## Findings Summary

| Label | Description |
| - | - |
|L-01| primitiveOutputAmount() dust outputAmount residue in Curve2PoolAdapter|
|L-02|_erc721Unwrap() Anyone can borrow NFT for free for voting/NFT verification|
|L-03|Use `isApprovedForAll` permission to restrict `forwardedDoInteraction()`, posing a significant financial risk to users|
|L-04|approveToken() If the token is USDT, it may not be executable|

## [L-01] primitiveOutputAmount() dust outputAmount residue in Curve2PoolAdapter
In `Curve2PoolAdapter.primitiveOutputAmount()` 
When `outputToken`'s decimals > 18, it will be converted to `18`, and the truncated `dust` will remain in the contract forever, never to be taken out.
If accumulated over time, it may lock up a considerable amount of residual `token` in the contract. 
It is suggested to add a method for administrators to take away these `dust`.

```solidity
    function primitiveOutputAmount(
        uint256 inputToken,
        uint256 outputToken,
        uint256 inputAmount,
        bytes32 minimumOutputAmount
    )
        internal
        override
        returns (uint256 outputAmount)
    {
...
@>      outputAmount = _convertDecimals(decimals[outputToken], NORMALIZED_DECIMALS, rawOutputAmount);

        if (uint256(minimumOutputAmount) > outputAmount) revert SLIPPAGE_LIMIT_EXCEEDED();

        if (action == ComputeType.Swap) {
            emit Swap(inputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);
        } else if (action == ComputeType.Deposit) {
            emit Deposit(inputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);
        } else {
            emit Withdraw(outputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);
        }
    }
```

## [L-02]_erc721Unwrap() Anyone can borrow NFT for free for voting/NFT verification and other applications that require NFT identity

Some NFTs will be used for identity verification. 
Currently, anyone can borrow NFT (through ERC721Unwarp and then ERC721warp) and perform identity verification, etc., in the custom `onERC1155Received()`. 
It is suggested to verify the real ownership of NFT when Unwarp.

```diff
    function _erc721Unwrap(address tokenAddress, uint256 tokenId, address userAddress, uint256 oceanId) private {

+       require(balanceOf(userAddress,oceanId)==1 ,"Not Your NFT");
        IERC721(tokenAddress).safeTransferFrom(address(this), userAddress, tokenId);
        emit Erc721Unwrap(tokenAddress, tokenId, userAddress, oceanId);
    }
```

## [L-03] Use `isApprovedForAll` permission to restrict `forwardedDoInteraction()`, posing a significant financial risk to users

Currently, authorized users (isApprovedForAll()==true) can operate other users' `Ocean:ERC1155` through `forwardedDoInteraction()`. 
But this user can also transfer the `token` that this user has not yet transferred into `Ocean:ERC1155` through `ERC20Wrap`. 
This is a very dangerous permission, far greater than just operating `Ocean:ERC1155` itself. 

For example:
Currently, `Alice` has 1000 `BTC`, and owns 1 `Ocean:ERC1155`, this `Ocean:ERC1155` corresponds to 1 BTC.
1. `Alice` executes `Ocean:ERC1155.setApprovalForAll(bob)`, mistakenly thinking that `bob` can only operate `Ocean:ERC1155`, which is worth `1 btc`.
2. But `bob` can transfer the `1000` `BTC` in `alice`'s wallet into `Ocean:ERC1155` through `forwardedDoInteraction()->ERC20Wrap`.
3. `bob` then transfers away `1001` `Ocean:ERC1155`.

The price `bob` can operate is far greater than `1 btc`.

It is suggested to add other permissions, do not use ERC1155's `isApprovedForAll()`. For example, add a new permission: `ApprovedOcean`, and note the risk in the document, this can operate any funds in the user's wallet.

## [L-04]_approveToken() If the token is USDT, it may not be executable
Currently, `Curve2PoolAdapter._approveToken` is executed using `IERC20Metadata.approve()`. 
But similar to `USDT`, the `approve()` method definition does not return. 
This leads to the use of `IERC20Metadata.approve()` will always `revert`.

It is recommended to use oz's `safeApprove()`.

