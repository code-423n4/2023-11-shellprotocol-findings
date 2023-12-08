### Report 1:
#### Absence of Reentrancy Guard for Erc20 Interactions
- As noted from the code provided below from the Ocean.sol contract only ERC1155 & ERC721 has the Reentrancy Guard, It should also be done for ERC20 Interactions too to ensure good security of the Ocean Protocol.
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L104-L109
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L171-L172
```solidity
  /// @dev Determines if a transfer callback is expected.
    /// @dev adapted from OpenZeppelin Reentrancy Guard
    uint256 constant NOT_INTERACTION = 1;
    uint256 constant INTERACTION = 2;
    uint256 _ERC1155InteractionStatus;
    uint256 _ERC721InteractionStatus;
 ...
   constructor(string memory uri_) OceanERC1155(uri_) {
        unwrapFeeDivisor = type(uint256).max;
>>>        _ERC1155InteractionStatus = NOT_INTERACTION;
>>>        _ERC721InteractionStatus = NOT_INTERACTION;
        WRAPPED_ETHER_ID = _calculateOceanId(address(0x4574686572), 0); // hexadecimal(ascii("Ether"))
    }
```
###  Report 2:
