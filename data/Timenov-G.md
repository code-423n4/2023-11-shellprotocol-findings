## Summary
[G-1] `INTERACTION` and `NOT_INTERACTION` can use different values.

### [G-1] `INTERACTION` and `NOT_INTERACTION` can use different values.
In `Ocean.sol` we are using the variables `INTERACTION` and `NOT_INTERACTION` to determine if a transfer callback is expected. These variables are of type `uint256` and hold the values of `1` and `2`. We can refactor a few lines and save some gas.

1. Change the variables values to 0 and 1

```diff
-    uint256 constant NOT_INTERACTION = 1;
-    uint256 constant INTERACTION = 2;
+    uint256 constant NOT_INTERACTION = 0;
+    uint256 constant INTERACTION = 1;
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L106-L107

2. Do not initialize `_ERC1155InteractionStatus` and `_ERC721InteractionStatus` in the constructor, because by default they will have value of `0` which will be `= NOT_INTERACTION`

```diff
    constructor(string memory uri_) OceanERC1155(uri_) {
        unwrapFeeDivisor = type(uint256).max;
-        _ERC1155InteractionStatus = NOT_INTERACTION;
-        _ERC721InteractionStatus = NOT_INTERACTION;
        WRAPPED_ETHER_ID = _calculateOceanId(address(0x4574686572), 0); // hexadecimal(ascii("Ether"))
    }
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L169-L174

3. Use `delete` instead of setting `_ERC721InteractionStatus` and `_ERC1155InteractionStatus` to `0`

```diff
-   _ERC721InteractionStatus = NOT_INTERACTION;
+   delete _ERC721InteractionStatus;
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L892

```diff
-   _ERC1155InteractionStatus = NOT_INTERACTION;
+   delete _ERC1155InteractionStatus;
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L932