## Gas Optimizations

| Number                                                                 | Issue                                                    | Instances |
| ---------------------------------------------------------------------- | :------------------------------------------------------- | :-------: |
| [[G-01](#g-01-state-variables-can-be-packed-into-fewer-storage-slots)] | `State variables` can be packed into fewer storage slots |     2     |
| [[G-02](#g-02-make-variable-outside-of-the-loop)]                      | Make variable outside of the loop                        |     8     |
| [[G-03](#g-03-dont-cache-any-value-if-it-is-used-only-once)]           | Don't `cache` any value if it is used only once          |     4     |
| [[G-04](#g-04-check-amount-for-zero-before-transferring)]              | Check `amount` for `zero` before transferring            |     1     |
| [[G-05](#g-05-use-named-returns-value-in-pure-functions)]              | Use named returns value in `pure` functions              |     4     |

## [G-01] `State variables` can be packed into fewer storage slots.

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this
will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the
variables packed together are retrieved together in functions, we will effectively save ~2000 gas with every subsequent
SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset
(20000 gas). Reads of the variables can also be cheaper

_2 Instances in 1 File_

### Reduce uint type for `_ERC1155InteractionStatus` and `_ERC721InteractionStatus` to `uint8` and can be packed together to save 1 SLOT (~2000 Gas)

Since `_ERC1155InteractionStatus` and `_ERC721InteractionStatus` used only to store 2 constant values `NOT_INTERACTION`
and `INTERACTION` whose values are **1** and **2** respectively. So `uint8` is more than sufficient to hold value if it
is 1 Or 2. We can see in entire contract `Ocean.sol` either `NOT_INTERACTION` Or `INTERACTION` assigned to both those
state variables. So it will save 1 SLOT by reducing the size of `_ERC1155InteractionStatus` and
`_ERC721InteractionStatus` to `uint8` and pack them into 1 SLOT.

```solidity
File : src/ocean/Ocean.sol


108:      uint256 _ERC1155InteractionStatus; //@audit reduce these to uint8
109:      uint256 _ERC721InteractionStatus; //@audit reduce these to uint8

```

[108-109](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L108-L109)

```diff
File : src/ocean/Ocean.sol

-       uint256 _ERC1155InteractionStatus;
-       uint256 _ERC721InteractionStatus;
+       uint8 _ERC1155InteractionStatus;
+       uint8 _ERC721InteractionStatus;
```

## [G-02] Make variable outside of the loop

Declaring variables outside of a loop can help save gas in Solidity by reducing gas consumption because it prevents
re-declaration of the variable in each iteration of the loop. When variables are declared outside the loop, the gas cost
associated with declaration occurs only once, outside the loop's scope.

_8 Instances in 1 File_

```solidity
File : src/ocean/Ocean.sol

501:      for (uint256 i = 0; i < interactions.length;) {
502:                interaction = interactions[i];
503:
504:                (InteractionType interactionType, address externalContract) =
506:                    _unpackInteractionTypeAndAddress(interaction);
507:
508:                // specifiedToken is the token whose amount the user specifies
509:                uint256 specifiedToken = _getSpecifiedToken(interactionType, externalContract, interaction);
...
514:                uint256 specifiedAmount;
515:                if (interaction.specifiedAmount == GET_BALANCE_DELTA) {
516:                    specifiedAmount = balanceDeltas.getBalanceDelta(interactionType, specifiedToken);
517:                } else {
518:                    specifiedAmount = interaction.specifiedAmount;
519:                }
520:
521:                (uint256 inputToken, uint256 inputAmount, uint256 outputToken, uint256 outputAmount) =
522:                _executeInteraction(
523:                    interaction, interactionType, externalContract, specifiedToken, specifiedAmount, userAddress_
524:                );

```

[501-524](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L501C13-L524C19)

```diff
File : src/ocean/Ocean.sol

+       InteractionType interactionType;
+       address externalContract;
+       uint256 specifiedToken;
+       uint256 specifiedAmount;
+       uint256 inputToken;
+       uint256 inputAmount;
+       uint256 outputToken;
+       uint256 outputAmount
        for (uint256 i = 0; i < interactions.length;) {
                interaction = interactions[i];

-               (InteractionType interactionType, address externalContract) =
-                   _unpackInteractionTypeAndAddress(interaction);
+               (interactionType, externalContract) =
+                   _unpackInteractionTypeAndAddress(interaction);


-               uint256 specifiedToken = _getSpecifiedToken(interactionType, externalContract, interaction);
+               specifiedToken = _getSpecifiedToken(interactionType, externalContract, interaction);

-               uint256 specifiedAmount;
+               specifiedAmount;
                if (interaction.specifiedAmount == GET_BALANCE_DELTA) {
                    specifiedAmount = balanceDeltas.getBalanceDelta(interactionType, specifiedToken);
                } else {
                    specifiedAmount = interaction.specifiedAmount;
                }

-               (uint256 inputToken, uint256 inputAmount, uint256 outputToken, uint256 outputAmount) =
+               (inputToken, inputAmount, outputToken, outputAmount) =
                _executeInteraction(
                    interaction, interactionType, externalContract, specifiedToken, specifiedAmount, userAddress_
                );

```

## [G-03] Don't `cache` any value if it is used only once

When variable used only once there is no need to cache it. It wastes extra gas of creating stack variable and writing
value to it.

_4 Instances in 3 Files_

```solidity
File : src/ocean/Ocean.sol

867:    uint256 amountRemaining = amount - feeCharged; //@audit don't cache amount - feeCharged

```

[867](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L867C13-L867C59)

```solidity
File : src/adapters/Curve2PoolAdapter.sol

103:     address tokenAddress = underlying[tokenId]; //@audit don't cache underlying[tokenId]

122:     address tokenAddress = underlying[tokenId]; //@audit don't cache underlying[tokenId]

```

[103](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L103C9-L103C52),
[122](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L122C9-L122C52)

```solidity
File : src/adapters/OceanAdapter.sol

71:      uint256 unwrappedAmount = inputAmount - unwrapFee; //@audit don't cache inputAmount - unwrapFee

```

[71](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L71C9-L71C59)

## [G-04] Check `amount` for `zero` before transferring

Checking if a value is zero before executing a transfer in Ethereum smart contracts can help prevent unnecessary
operations and potentially save gas. And transferring zero amount doesn't change anything and waste gas. **Note: These
instance missed by bot-report**

_1 Instances in 1 File_

```solidity
File : src/ocean/Ocean.sol

982:    payable(userAddress).transfer(transferAmount); //@audit check transferAmount for 0

```

[982](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L982C8-L982C55)

## [G-05] Use named returns value in `pure` functions

### Proof of Concept

```solidity
library NoNamedReturnArithmetic {
  function sum(uint256 num1, uint256 num2) internal pure returns (uint256) {
    return num1 + num2;
  }
}

contract NoNamedReturn {
  using NoNamedReturnArithmetic for uint256;

  uint256 public stateVar;

  function add2State(uint256 num) public {
    stateVar = stateVar.sum(num);
  }
}
```

```solidity
test for test/NoNamedReturn.t.sol:NamedReturnTest
[PASS] test_Increment() (gas: 27639)
```

```solidity
library NamedReturnArithmetic {
  function sum(uint256 num1, uint256 num2) internal pure returns (uint256 theSum) {
    theSum = num1 + num2;
  }
}

contract NamedReturn {
  using NamedReturnArithmetic for uint256;

  uint256 public stateVar;

  function add2State(uint256 num) public {
    stateVar = stateVar.sum(num);
  }
}
```

```solidity
test for test/NamedReturn.t.sol:NamedReturnTest
[PASS] test_Increment() (gas: 27613)
```

_4 Instances in 2 Files_

```solidity
File : src/ocean/Ocean.sol

353:    function onERC1155BatchReceived(
354:          address,
355:          address,
356:          uint256[] calldata,
357:          uint256[] calldata,
358:          bytes calldata
359:     )
360:          external
361:          pure
362:          override
363:          returns (bytes4)
364:     {
365:         return 0;
366:     }

```

[353-366](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L353C4-L366C6)

```diff
File : src/ocean/Ocean.sol

     function onERC1155BatchReceived(
         address,
         address,
         uint256[] calldata,
         uint256[] calldata,
         bytes calldata
     )
         external
         pure
         override
-        returns (bytes4)
+        returns (bytes4 zero)
    {
-        return 0;
+        zero = bytes4(0);
    }

```

```solidity
File : src/adapters/OceanAdapter.sol

99:     function _fetchInteractionId(address token, uint256 interactionType) internal pure returns (bytes32) {
100:         uint256 packedValue = uint256(uint160(token));
101:         packedValue |= interactionType << 248;
102:         return bytes32(abi.encode(packedValue));
103:     }

```

[99-103](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L99C1-L103C6)

```diff
File : src/adapters/OceanAdapter.sol

-     function _fetchInteractionId(address token, uint256 interactionType) internal pure returns (bytes32) {
+     function _fetchInteractionId(address token, uint256 interactionType) internal pure returns (bytes32 _packedValue) {
        uint256 packedValue = uint256(uint160(token));
        packedValue |= interactionType << 248;
-       return bytes32(abi.encode(packedValue));
+       _packedValue = bytes32(abi.encode(packedValue));
    }

```

```solidity
File : src/adapters/OceanAdapter.sol

108:    function _calculateOceanId(address tokenAddress, uint256 tokenId) internal pure returns (uint256) {
109:        return uint256(keccak256(abi.encodePacked(tokenAddress, tokenId)));
110:    }

```

[108-110](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L108C1-L110C6)

```diff
File : src/adapters/OceanAdapter.sol

-      function _calculateOceanId(address tokenAddress, uint256 tokenId) internal pure returns (uint256) {
-          return uint256(keccak256(abi.encodePacked(tokenAddress, tokenId)));
+      function _calculateOceanId(address tokenAddress, uint256 tokenId) internal pure returns (uint256 _tokenId) {
+          _tokenId = uint256(keccak256(abi.encodePacked(tokenAddress, tokenId)));
    }

```

```solidity
File : src/adapters/OceanAdapter.sol

117:     function onERC1155Received(address, address, uint256, uint256, bytes memory) public pure returns (bytes4) {
118:         return this.onERC1155Received.selector;
119:     }

```

[117-119](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L117C1-L119C6)

```diff
File : src/adapters/OceanAdapter.sol

-      function onERC1155Received(address, address, uint256, uint256, bytes memory) public pure returns (bytes4) {
-          return this.onERC1155Received.selector;
+      function onERC1155Received(address, address, uint256, uint256, bytes memory) public pure returns (bytes4 _selector) {
+          _selector = this.onERC1155Received.selector;
    }

```

