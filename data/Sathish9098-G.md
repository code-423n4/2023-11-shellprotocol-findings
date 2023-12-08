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

##

## [G-2] Don't cache state variable only used once

There are a few instances where state variables are assigned to local variables but are only used once. These can be optimized by directly using the state variable in the expression.

```diff
FILE: 2023-11-shellprotocol/src/adapters/Curve2PoolAdapter.sol

function wrapToken(uint256 tokenId, uint256 amount) internal override {
-        address tokenAddress = underlying[tokenId];

        Interaction memory interaction = Interaction({
-            interactionTypeAndAddress: _fetchInteractionId(tokenAddress, 
+            interactionTypeAndAddress: _fetchInteractionId(underlying[tokenId], uint256(InteractionType.WrapErc20)),
            inputToken: 0,
            outputToken: 0,
            specifiedAmount: amount,
            metadata: bytes32(0)
        });

        IOceanInteractions(ocean).doInteraction(interaction);
    }

``` 
https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L102-L114

##

## [G-3] Modulus operations that could be unchecked

Modulus operations should be unchecked to save gas since they cannot overflow or underflow. Execution of modulus operations outside unchecked blocks adds nothing but overhead. Saves about 30 gas.

```solidity
FILE: 2023-11-shellprotocol/src/ocean/Ocean.sol

1145:  truncatedAmount = amountToConvert % shift;

```
https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L1145

##

## [G-4] Use assembly to validate msg.sender

We can use assembly to efficiently validate msg.sender with the least amount of opcodes necessary. For more details check the following report [Here](https://code4rena.com/reports/2023-05-juicebox#g-06-use-assembly-to-validate-msgsender)

```solidity
FILE : 2023-11-shellprotocol/src/adapters/OceanAdapter.sol

39: require(msg.sender == ocean);

```
https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L39






