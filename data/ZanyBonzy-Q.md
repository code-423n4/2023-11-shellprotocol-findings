
1. Case against CurveTricryptoAdapter

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L25

The `CurveTricryptoAdapter` is the adapter that helps the Ocean integrate with the curve tricrypto pool. Hence calls are made from it to the Arbitrum tricrypto pool. However, due to an issue with the pool's current vulnerable vyper implementation. Ultimately, the Curve team has advised users to not use the tricrypto pool on arbitrum until a replacement is deployed. This issue stems from the `get_virtual_price()` function which is used to get the curve oracle price that is used to tweak prices for adding, swapping and removing of liquidity. Through this vulnerability, attackers can exploit the pool's functionality in unintended ways, leading to unexpected behavior or security risks. 

The reason given by the team was that this contract uses a vulnerable version of vyper and although this pool uses WETH and not ETH, an attacker can still halt operations on this pool.

A [breakdown](https://rekt.news/curve-vyper-rekt/) of the issue. Curve Finance's [tweet1](https://twitter.com/CurveFinance/status/1685933800088391680)/[tweet2](https://twitter.com/CurveFinance/status/1692170455598465341) and as of time of audit, there has been no update about resolution of the tricrypto pool's issues. Therefore, we recommend not interacting with the pool, for the safety of the users until the problem is fixed as interacting with the pool would be knowingly putting the users' funds at risk.


2. Protocol assumes a 1:1 ETH/WETH ratio

When wrapping ether
```
        if (msg.value != 0) {
            inputToken = 0;
            inputAmount = 0;
            outputToken = WRAPPED_ETHER_ID;
            outputAmount = msg.value;
            emit EtherWrap(msg.value, userAddress);
        } 
```
and unwrapping eth,
```
    function _etherUnwrap(uint256 amount, address userAddress) private {
        uint256 feeCharged = _calculateUnwrapFee(amount);
        _grantFeeToOcean(WRAPPED_ETHER_ID, feeCharged);
        uint256 transferAmount = amount - feeCharged;
        payable(userAddress).transfer(transferAmount);
        emit EtherUnwrap(transferAmount, feeCharged, userAddress);
    }
```

the protocol implicitly implies a 1:1 ratio, however real prices of WETH to ETH is not always 1 to 1. Currently, WETH/ETH trades at about 0.9976 and from [historical](https://www.coingecko.com/en/coins/weth) data, WETH has been worth as high as 3.6 ETH and as low as 0.7. This can lead to a host of accounting and swapping issues, especially during integrations with the tricrypto pool (It uses a `tweak_price` function to get prices from oracles) as users might get more or less than they deserve during transactions.


3. Users can use the the [backwards compatibility](https://eips.ethereum.org/EIPS/eip-1155) property of ERC1155 with ERC721 to bypass paying unwrapFees on hybrid ERC-1155/ERC-721 contracts (e.g., https://github.com/thesandboxgame/thesandbox-contracts/blob/master/src/Asset/ERC1155ERC721.sol). Users can wrap in bulk the NFTs as erc1155,
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
        if (tokenAddress == address(this)) revert NO_RECURSIVE_WRAPS();
        _ERC1155InteractionStatus = INTERACTION;
        IERC1155(tokenAddress).safeTransferFrom(userAddress, address(this), tokenId, amount, "");
        _ERC1155InteractionStatus = NOT_INTERACTION;
        emit Erc1155Wrap(tokenAddress, tokenId, amount, userAddress, oceanId);
    }
```
 and individually unwrap each one as erc721 at no c0st.
  ```
   function _erc721Unwrap(address tokenAddress, uint256 tokenId, address userAddress, uint256 oceanId) private {
        IERC721(tokenAddress).safeTransferFrom(address(this), userAddress, tokenId);
        emit Erc721Unwrap(tokenAddress, tokenId, userAddress, oceanId);
    }
```
4. The return value of .transfer is not checked. It is usually good to add a require-statement that checks the return value or to use something like safeTransfer; unless one is sure the given token reverts in case of a failure.

```
        payable(userAddress).transfer(transferAmount);
        emit EtherUnwrap(transferAmount, feeCharged, userAddress);
    }

```
The transfer could fail and as its result is not being verified, the unwrap will not succeed and the user will lose all of the token. Consider adding a require statement to check for transfer success.
5. WETH might not be received when swapping liquidity in the tricrypto

Curve tricrypto pool conducts transactions with WETH and unless the `useEth` is explicitly set to `True`. Otherwise, it will default to `False` 
```
def exchange(i: uint256, j: uint256, dx: uint256, min_dy: uint256, use_eth: bool = False) -> uint256:
    assert not self.is_killed  # dev: the pool is killed
    assert i != j  # dev: coin index out of range
    assert i < N_COINS  # dev: coin index out of range
    assert j < N_COINS  # dev: coin index out of range
    assert dx > 0  # dev: do not exchange 0 coins

    A_gamma: uint256[2] = self._A_gamma()
    xp: uint256[N_COINS] = self.balances
    ix: uint256 = j
    p: uint256 = 0
    dy: uint256 = 0
```


The `CurveTricryptoAdapter` function swaps tokens and weth using the exchange function in the tricrypto primitive and in doing this sets the `useEth` parameter to true. That is, the tricrypto swap transaction uses `ETH` instead of the protocol's own `WETH`, this is unnecessary as the tricrypto pool will have to unwrap the protcol's `WETH` to `ETH` before performming the swaps. In a case when the outputToken is the zToken, i.e user should recieve `WETH`, the returned token from the swap is `ETH` and not `WETH` as required by the protocol. 
      
      ```
        if (action == ComputeType.Swap) {
            bool useEth = inputToken == zToken || outputToken == zToken;

            ICurveTricrypto(primitive).exchange{ value: inputToken == zToken ? rawInputAmount : 0 }(
                indexOfInputAmount, indexOfOutputAmount, rawInputAmount, 0, useEth //@note potocol uses weth
            );
            ```


