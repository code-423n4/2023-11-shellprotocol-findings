### [Low-0] Unnecessary use of `_ERC721InteractionStatus` & `_ERC1155InteractionStatus` before and after respectively while transfering ERC721 & ERC1155 as there is no check(modifier or internal function) to validate this status.

These uses only increases gas cost by writing to storage.

```diff
        _ERC1155InteractionStatus = INTERACTION; 
        IERC1155(tokenAddress).safeTransferFrom(userAddress, address(this), tokenId, amount, "");
        _ERC1155InteractionStatus = NOT_INTERACTION;
        emit Erc1155Wrap(tokenAddress, tokenId, amount, userAddress, oceanId);
```
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L930-L932
```
```diff
        _ERC721InteractionStatus = INTERACTION; 
        IERC721(tokenAddress).safeTransferFrom(userAddress, address(this), tokenId);
        _ERC721InteractionStatus = NOT_INTERACTION;
```
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L890-L892
```

### [Low-0] Event is emited with wrong `amount` value
In `_erc1155Unwrap()` function event emited with 
```solidity
emit Erc1155Unwrap(tokenAddress, tokenId, amount, feeCharged, userAddress, oceanId);
```
```
where amount = amount passed to function
```

While in `_etherUnwrap()` function event emited with
```solidity
emit EtherUnwrap(transferAmount, feeCharged, userAddress);
```
```
Where transferAmount = amount - feeCharged
```
As per code base comment
```
* @notice unwrap amounts may be subject to a fee that reduces the amount
     *  moved on the external token's ledger. To unwrap an exact amount, the
     *  caller should compute off-chain what specifiedAmount results in the
     *  desired unwrap amount. If the user wants to receive 100_000 of a token
     *  and the fee is 1 basis point, the user should specify 100_010
     *  This value was found by solving for x in the equation:
     *      x - Floor[x * (1/10000)] = 100000
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L943-L949

So event emited in case of `_erc1155Unwrap` is wrong, it should emited with 
amount = amount - feeCharged; 

```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L969
```

