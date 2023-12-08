## Table of Contents

- [Table of Contents](#table-of-contents)
- [Pack variables together(Save 1 SLOT: 2100 Gas)](#pack-variables-togethersave-1-slot-2100-gas)
- [No need to assign default values to struct(not similar to what the bot reported)](#no-need-to-assign-default-values-to-structnot-similar-to-what-the-bot-reported)


## Pack variables together(Save 1 SLOT: 2100 Gas)
https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L108-L109
```solidity
File: /src/ocean/Ocean.sol
108:    uint256 _ERC1155InteractionStatus;
109:    uint256 _ERC721InteractionStatus;
```

The above two variables are set in the following 
https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L169-L174
```solidity
169:    constructor(string memory uri_) OceanERC1155(uri_) {
170:        unwrapFeeDivisor = type(uint256).max;
171:        _ERC1155InteractionStatus = NOT_INTERACTION;
172:        _ERC721InteractionStatus = NOT_INTERACTION;
173:        WRAPPED_ETHER_ID = _calculateOceanId(address(0x4574686572), 0); // hexadecimal(ascii("Ether"))
174:    }
```

Note, we are assigning them values of `NOT_INTERACTION`  which is a constant variable with value `1`. The other place we assign them is in the function `_erc1155Wrap()` where we now use the variable `INTERACTION` which is a constant variable with value `1`
We can safely reduce their size from `uint256` to `uint128` since the only two values we ever set are `1` and `2`


```diff
diff --git a/src/ocean/Ocean.sol b/src/ocean/Ocean.sol
index 1b687be..d3ac600 100644
--- a/src/ocean/Ocean.sol
+++ b/src/ocean/Ocean.sol
@@ -105,8 +105,8 @@ contract Ocean is IOceanInteractions, IOceanFeeChange, OceanERC1155, IERC721Rece
     /// @dev adapted from OpenZeppelin Reentrancy Guard
     uint256 constant NOT_INTERACTION = 1;
     uint256 constant INTERACTION = 2;
-    uint256 _ERC1155InteractionStatus;
-    uint256 _ERC721InteractionStatus;
+    uint128 _ERC1155InteractionStatus;
+    uint128 _ERC721InteractionStatus;

     event ChangeUnwrapFee(uint256 oldFee, uint256 newFee, address sender);
     event Erc20Wrap(
```


## No need to assign default values to struct(not similar to what the bot reported)
https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L118-L140
```solidity
File: /src/adapters/CurveTricryptoAdapter.sol
118:    function wrapToken(uint256 tokenId, uint256 amount) internal override {
119:        Interaction memory interaction;

121:        if (tokenId == zToken) {
122:            interaction = Interaction({
123:                interactionTypeAndAddress: 0,
124:                inputToken: 0,
125:                outputToken: 0,
126:                specifiedAmount: 0,
127:                metadata: bytes32(0)
128:            });
129:            IOceanInteractions(ocean).doInteraction{ value: amount }(interaction);
130:        } else {
131:            interaction = Interaction({
132:                interactionTypeAndAddress: _fetchInteractionId(underlying[tokenId], uint256(InteractionType.WrapErc20)),
133:                inputToken: 0,
134:                outputToken: 0,
135:                specifiedAmount: amount,
136:                metadata: bytes32(0)
137:            });
138:            IOceanInteractions(ocean).doInteraction(interaction);
139:        }
140:    }
```

We are defining a variable of type `Interaction` which is a struct. At no point is this struct being stored in storage so any values assigned only last inside the blocks they are assigned. This means that , every time the struct is being used, we start off at the default values of all variables in the struct, there is no need therefore to assign the same default value

```diff
@@ -119,22 +119,11 @@ contract CurveTricryptoAdapter is OceanAdapter {
         Interaction memory interaction;

         if (tokenId == zToken) {
-            interaction = Interaction({
-                interactionTypeAndAddress: 0,
-                inputToken: 0,
-                outputToken: 0,
-                specifiedAmount: 0,
-                metadata: bytes32(0)
-            });
             IOceanInteractions(ocean).doInteraction{ value: amount }(interaction);
         } else {
-            interaction = Interaction({
-                interactionTypeAndAddress: _fetchInteractionId(underlying[tokenId], uint256(InteractionType.WrapErc20)),
-                inputToken: 0,
-                outputToken: 0,
-                specifiedAmount: amount,
-                metadata: bytes32(0)
-            });
+            interaction.interactionTypeAndAddress = _fetchInteractionId(underlying[tokenId], uint256(InteractionType.WrapErc20));
+            interaction.specifiedAmount = amount;
+
             IOceanInteractions(ocean).doInteraction(interaction);
         }
     }
```



https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L147-L169
```solidity
File: /src/adapters/CurveTricryptoAdapter.sol
147:    function unwrapToken(uint256 tokenId, uint256 amount) internal override {
148:        Interaction memory interaction;

150:        if (tokenId == zToken) {
151:            interaction = Interaction({
152:                interactionTypeAndAddress: _fetchInteractionId(address(0), uint256(InteractionType.UnwrapEther)),
153:                inputToken: 0,
154:                outputToken: 0,
155:                specifiedAmount: amount,
156:                metadata: bytes32(0)
157:            });
158:        } else {
159:            interaction = Interaction({
160:                interactionTypeAndAddress: _fetchInteractionId(underlying[tokenId], uint256(InteractionType.UnwrapErc20)),
161:                inputToken: 0,
162:                outputToken: 0,
163:                specifiedAmount: amount,
164:                metadata: bytes32(0)
165:            });
166:        }

168:        IOceanInteractions(ocean).doInteraction(interaction);
169:    }
```

```diff
      * @dev unwraps the underlying token from the Ocean
@@ -148,21 +148,13 @@ contract CurveTricryptoAdapter is OceanAdapter {
         Interaction memory interaction;

         if (tokenId == zToken) {
-            interaction = Interaction({
-                interactionTypeAndAddress: _fetchInteractionId(address(0), uint256(InteractionType.UnwrapEther)),
-                inputToken: 0,
-                outputToken: 0,
-                specifiedAmount: amount,
-                metadata: bytes32(0)
-            });
+                interaction.interactionTypeAndAddress = _fetchInteractionId(address(0), uint256(InteractionType.UnwrapEther));
+                interaction.specifiedAmount = amount;
+
         } else {
-            interaction = Interaction({
-                interactionTypeAndAddress: _fetchInteractionId(underlying[tokenId], uint256(InteractionType.UnwrapErc20)),
-                inputToken: 0,
-                outputToken: 0,
-                specifiedAmount: amount,
-                metadata: bytes32(0)
-            });
+                interaction.interactionTypeAndAddress = _fetchInteractionId(underlying[tokenId], uint256(InteractionType.UnwrapErc20));
+                interaction.specifiedAmount = amount;
+
         }

         IOceanInteractions(ocean).doInteraction(interaction);
```

**Similar scenarios**

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L102-L114
```solidity
File: /src/adapters/Curve2PoolAdapter.sol
102:    function wrapToken(uint256 tokenId, uint256 amount) internal override {
103:        address tokenAddress = underlying[tokenId];

105:        Interaction memory interaction = Interaction({
106:            interactionTypeAndAddress: _fetchInteractionId(tokenAddress, uint256(InteractionType.WrapErc20)),
107:            inputToken: 0,
108:            outputToken: 0,
109:            specifiedAmount: amount,
110:            metadata: bytes32(0)
111:        });

113:        IOceanInteractions(ocean).doInteraction(interaction);
114:    }
```


https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L121-L133
```solidity
File: /src/adapters/Curve2PoolAdapter.sol
121:    function unwrapToken(uint256 tokenId, uint256 amount) internal override {
122:        address tokenAddress = underlying[tokenId];

124:        Interaction memory interaction = Interaction({
125:            interactionTypeAndAddress: _fetchInteractionId(tokenAddress, uint256(InteractionType.UnwrapErc20)),
126:            inputToken: 0,
127:            outputToken: 0,
128:            specifiedAmount: amount,
129:            metadata: bytes32(0)
130:        });

132:        IOceanInteractions(ocean).doInteraction(interaction);
133:    }
```

