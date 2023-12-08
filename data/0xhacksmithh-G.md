### [Gas-01] Unnecessary Calculation

```diff
```
```
```

### [Gas-02] Could be initialized with `_ERC1155InteractionStatus` & `_ERC721InteractionStatus` Non-zero values

As In `constructor`, `_ERC1155InteractionStatus` & `_ERC721InteractionStatus` set to `NOT_INTERACTION` which equal to `1`. So This could do eairlier during declaration of these 2 variable, so gas cost will saved when storage write from 0 to non zero

```diff
-   uint256 _ERC1155InteractionStatus; 
-   uint256 _ERC721InteractionStatus;

+   uint256 _ERC1155InteractionStatus = NOT_INTERACTION; 
+   uint256 _ERC721InteractionStatus = NOT_INTERACTION;
```
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L108-L109
```

### [Gas-03] Instead of caching `_getSpecifiedToken()`, this could directly used inside follow up function `_executeInteraction()`

```diff
- uint256 specifiedToken = _getSpecifiedToken(interactionType, externalContract, interaction);

           
-            (inputToken, inputAmount, outputToken, outputAmount) = _executeInteraction(
-               interaction, interactionType, externalContract, specifiedToken, interaction.specifiedAmount, userAddress
            );

+ (inputToken, inputAmount, outputToken, outputAmount) = _executeInteraction(
                interaction, interactionType, externalContract, _getSpecifiedToken(interactionType, externalContract, interaction), interaction.specifiedAmount, userAddress
            );
```
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L405-L412
```

### [Gas-04] Initialize with non-zero number

```diff
```
```
```

### [Gas-05] `_idLength = ids.length;` should written one step above so that gas for extra `ids.length` calculation could saved

```diff
+        uint256 _idLength = ids.length;
+        BalanceDelta[] memory balanceDeltas = new BalanceDelta[](_idLength);

-        BalanceDelta[] memory balanceDeltas = new BalanceDelta[](ids.length); 

-        uint256 _idLength = ids.length;
```
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L462
```

### [Gas-06] Caching `ids.length` costs more gas than using as it is loops

```diff
    function _doMultipleInteractions(
        Interaction[] calldata interactions,
...
...
        uint256 _idLength = ids.length; 
        for (uint256 i = 0; i < _idLength;) {
            balanceDeltas[i] = BalanceDelta(ids[i], 0); 
            unchecked {
                ++i;
            }
        }
```
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L460-L463
```

### [Gas-07] No need to Cache `userAddress` to storage
As `userAddress` is a function argument no need to cache it, and it(cache value) only used 2 times in that block
```diff
address userAddress_ = userAddress;
```
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L499
```

### [Gas-08] Create Variable out side of loop and override it with each iteration
Total 6 variable created inside for loop in each iteration. So with each iteration
6 * (memory creation cost) gas will lost
```diff
+           uint256 inputToken;
+           uint256 inputAmount;
+           uint256 outputToken;
+           uint256 outputAmount;
+           uint256 specifiedToken
+           uint256 specifiedAmount;

            for (uint256 i = 0; i < interactions.length;) {
                interaction = interactions[i];

                (InteractionType interactionType, address externalContract) =
                    _unpackInteractionTypeAndAddress(interaction);

               
 -              uint256 specifiedToken = _getSpecifiedToken(interactionType, externalContract, interaction);
 +              specifiedToken = _getSpecifiedToken(interactionType, externalContract, interaction);

                
-               uint256 specifiedAmount;
                if (interaction.specifiedAmount == GET_BALANCE_DELTA) {
                    specifiedAmount = balanceDeltas.getBalanceDelta(interactionType, specifiedToken);
                } else {
                    specifiedAmount = interaction.specifiedAmount;
                }

 -              (uint256 inputToken, uint256 inputAmount, uint256 outputToken, uint256 outputAmount) =
                _executeInteraction(
                    interaction, interactionType, externalContract, specifiedToken, specifiedAmount, userAddress_
                );

 +              (inputToken, inputAmount, outputToken, outputAmount) =
                _executeInteraction(
                    interaction, interactionType, externalContract, specifiedToken, specifiedAmount, userAddress_
                );

```
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L508
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L514
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L521
```

### [Gas-09] `_calculateUnwrapFee()` could directly used for follow up calculation with-out caching it

```diff
-           uint256 feeCharged = _calculateUnwrapFee(amount); 
-           uint256 amountRemaining = amount - feeCharged;
 
+           uint256 amountRemaining = amount - _calculateUnwrapFee(amount);
```
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L866
```

### [Gas-10] Storage could be packed to save extra slot

```diff
-   uint256 _ERC1155InteractionStatus; 
-   uint256 _ERC721InteractionStatus;

+   uint128 _ERC1155InteractionStatus; 
+   uint128 _ERC721InteractionStatus;
```
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L108-L109
```


### [Gas-11] No Need To cache mathematical operation i.e `uint256 amountRemaining = amount - feeCharged`

```diff
-       uint256 amountRemaining = amount - feeCharged; 
        _grantFeeToOcean(oceanId, feeCharged);
-       IERC1155(tokenAddress).safeTransferFrom(address(this), userAddress, tokenId, amountRemaining, "");

+       IERC1155(tokenAddress).safeTransferFrom(address(this), userAddress, tokenId, amount - feeCharged, "");
```
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L966
```

### [Gas-12] `+= 1` Could replace with `unchecked {++}`

```diff
-       transferAmount += 1;

+       unchecked{ transferAmount++ };
```
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L1092
```

### [Gas-13] Some Substractions `-` could marked as uncheck due to previous condition check.

```diff
            assert(normalizedTransferAmount > amount);
-           dust = normalizedTransferAmount - amount;

+           dust = unchecked{ normalizedTransferAmount - amount };
```
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L1102
```

### [Gas-14] No Need to assign `0` to `dust` & `truncatedAmount` in functions calculation as it's default value already `0` 

```diff
        } else {
            dust = 0; // @audit G: leave
        }
```
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L1107
```
```diff
if (decimalsFrom == decimalsTo) {
            // no shift
            convertedAmount = amountToConvert;
            truncatedAmount = 0; // @audit G: leave truncate
        } else if (decimalsFrom < decimalsTo) {
            // Decimal shift left (add precision)
            uint256 shift = 10 ** (uint256(decimalsTo - decimalsFrom)); 
            convertedAmount = amountToConvert * shift;
            truncatedAmount = 0; // @audit
        } else {
            // Decimal shift right (remove precision) -> truncation
            uint256 shift = 10 ** (uint256(decimalsFrom - decimalsTo));
            convertedAmount = amountToConvert / shift;
            truncatedAmount = amountToConvert % shift; // @audit G: Uncheck
        }
```
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L1135
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L1140
```

### [Gas-15] Uncheck `%` operation 

```diff
truncatedAmount = amountToConvert % shift; // @audit G: Uncheck
```
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L1145
```

