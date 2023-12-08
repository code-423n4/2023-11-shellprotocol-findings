## [G-01] At the begining of function `_executeInteraction`, check `specifiedAmount` and return early when `specifiedAmount == 0`.
If `specifiedAmount` is 0, there is no need to do anything, return early to save gas.
```solidity
597:    function _executeInteraction(
598:        Interaction memory interaction,
599:        InteractionType interactionType,
600:        address externalContract,
601:        uint256 specifiedToken,
602:        uint256 specifiedAmount,
603:        address userAddress
604:    )
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L597-L604