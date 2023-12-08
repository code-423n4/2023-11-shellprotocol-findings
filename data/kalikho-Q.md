#
The `_mint(userAddress, outputToken, outputAmount)` function in `Ocean.sol` does not emit an event. 

Link to code in `Ocean.sol`: https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L428

Link to code in `OceanERC1155.sol`: https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/OceanERC1155.sol#L446

Due to the missing event the blockchain network will not be able to know about minting of new token.






