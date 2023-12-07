# GAS OPTIMIZATION

##

## [G-1] Optimizing state variables for minimal storage usage

The EVM uses 32-byte words, allowing variables smaller than 32 bytes to be packed together in one storage slot if their combined size is ≤32 bytes. Accessing these packed variables saves roughly 2000 gas per SLOAD, costing only 100 gas ('Gwarmaccess') compared to 2100 gas ('Gcoldsload') for separate slots.

### [G-1.1]  ``_ERC1155InteractionStatus`` , ``_ERC721InteractionStatus`` can be packed with same SLOT : Saves ``2000 GAS`` , ``1 SLOT``

The value of ``_ERC1155InteractionStatus`` and ``_ERC721InteractionStatus`` values always interchanged between ``NOT_INTERACTION`` , ``INTERACTION `` constant values . So uint128 perfectly safe instead of uint256. This will not harm protocol and perfectly safe conversion.

```diff
FILE: 2023-11-shellprotocol/src/ocean/Ocean.sol

106: uint256 constant NOT_INTERACTION = 1;
107: uint256 constant INTERACTION = 2;
- 108: uint256 _ERC1155InteractionStatus;
- 109: uint256 _ERC721InteractionStatus;
+ 108: uint128 _ERC1155InteractionStatus;
+ 109: uint128 _ERC721InteractionStatus;

```
https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L108-L109