## [G-1] Using XOR (^) and OR (|) bitwise equivalents

Estimated savings: 73 gas
Max savings according to yarn profile: 282 gas

On Remix, given only uint256 types, the following are logical equivalents, but don’t cost the same amount of gas:

(a != b || c != d || e != f) costs 571
((a ^ b) | (c ^ d) | (e ^ f)) != 0 costs 498 (saving 73 gas)
Consider rewriting as following to save gas:

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.
Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.

Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas:

However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values

There are 3 instances of this issue in 2 files:

```
File: Ocean.sol	

709: if (interactionType == InteractionType.WrapErc20 || interactionType == InteractionType.UnwrapErc20) {    

712: interactionType == InteractionType.WrapErc721 || interactionType == InteractionType.WrapErc1155
713:     || interactionType == InteractionType.UnwrapErc721 || interactionType == InteractionType.UnwrapErc1155
```

    diff --git a/src/ocean/Ocean.sol b/src/ocean/Ocean.sol
    index 1b687be..a3a3436 100644
    --- a/src/ocean/Ocean.sol
    +++ b/src/ocean/Ocean.sol
    @@ -706,11 +706,11 @@ contract Ocean is IOceanInteractions, IOceanFeeChange, OceanERC1155, IERC721Rece
             view
             returns (uint256 specifiedToken)
         {
    -        if (interactionType == InteractionType.WrapErc20 || interactionType == InteractionType.UnwrapErc20) {
    +        if (((interactionType ^ InteractionType.WrapErc20) & (interactionType ^ InteractionType.UnwrapErc20)) ==  0 ) {
                 specifiedToken = _calculateOceanId(externalContract, 0);
             } else if (
    -            interactionType == InteractionType.WrapErc721 || interactionType == InteractionType.WrapErc1155
    -                || interactionType == InteractionType.UnwrapErc721 || interactionType == InteractionType.UnwrapErc1155
    +            ((interactionType ^ InteractionType.WrapErc721) & (interactionType ^ InteractionType.WrapErc1155)
    +                & (interactionType ^ InteractionType.UnwrapErc721) & (interactionType ^ InteractionType.UnwrapErc1155)) == 0
             ) {
                 specifiedToken = _calculateOceanId(externalContract, uint256(interaction.metadata));
             } else if (interactionType == InteractionType.ComputeInputAmount) {

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol

```
File: CurveTricryptoAdapter.sol	
 
199: bool useEth = inputToken == zToken || outputToken == zToken;
```

    diff --git a/src/adapters/CurveTricryptoAdapter.sol b/src/adapters/CurveTricryptoAdapter.sol
    index 948169f..dbe2491 100644
    --- a/src/adapters/CurveTricryptoAdapter.sol
    +++ b/src/adapters/CurveTricryptoAdapter.sol
    @@ -196,7 +196,7 @@ contract CurveTricryptoAdapter is OceanAdapter {
             uint256 indexOfOutputAmount = indexOf[outputToken];

             if (action == ComputeType.Swap) {
    -            bool useEth = inputToken == zToken || outputToken == zToken;
    +            bool useEth = ((inputToken ^ zToken) & (outputToken ^ zToken)) == 0;

                 ICurveTricrypto(primitive).exchange{ value: inputToken == zToken ? rawInputAmount : 0 }(
                     indexOfInputAmount, indexOfOutputAmount, rawInputAmount, 0, useEth

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(5,10,16);
            c1.optimized(5,10,16);
        }
    }
    contract Contract0 {
        address a;
        function not_optimized(uint8 x,uint8 y, uint8 z) public returns(bool){
            if(x!=5 || y!=10 || z!=15){
                return true;
            }
        }
    }
    contract Contract1 {
        address a;
        function optimized(uint8 x,uint8 y, uint8 z) public returns(bool){
            if(((x^5) | (y^10) | (z^15)) != 0){
                return true;
            }
        }
    }
    

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 53705                                     | 300             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 622             | 622 | 622    | 622 | 1       |

| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 49899                                     | 280             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 569             | 569 | 569    | 569 | 1       |

## [G-02] Stack variable used as a cheaper cache for a state variable is only used once

If the variable is only accessed once, it’s cheaper to use the state variable directly that one time, and save the 3 gas the extra stack assignment would spend.

There are 5 instances of this issue in 3 files:

```
File: Ocean.sol	

867: uint256 amountRemaining = amount - feeCharged;

966: uint256 amountRemaining = amount - feeCharged;
```

    diff --git a/src/ocean/Ocean.sol b/src/ocean/Ocean.sol
    index 1b687be..2104164 100644
    --- a/src/ocean/Ocean.sol
    +++ b/src/ocean/Ocean.sol
    @@ -864,10 +864,9 @@ contract Ocean is IOceanInteractions, IOceanFeeChange, OceanERC1155, IERC721Rece
         function _erc20Unwrap(address tokenAddress, uint256 amount, address userAddress, uint256 inputToken) private {
             try IERC20Metadata(tokenAddress).decimals() returns (uint8 decimals) {
                 uint256 feeCharged = _calculateUnwrapFee(amount);
    -            uint256 amountRemaining = amount - feeCharged;

                 (uint256 transferAmount, uint256 truncated) =
    -                _convertDecimals(NORMALIZED_DECIMALS, decimals, amountRemaining);
    +                _convertDecimals(NORMALIZED_DECIMALS, decimals, amount - feeCharged);
                 feeCharged += truncated;

                 _grantFeeToOcean(inputToken, feeCharged);
    @@ -963,9 +962,8 @@ contract Ocean is IOceanInteractions, IOceanFeeChange, OceanERC1155, IERC721Rece
         {
             if (tokenAddress == address(this)) revert NO_RECURSIVE_UNWRAPS();
             uint256 feeCharged = _calculateUnwrapFee(amount);
    -        uint256 amountRemaining = amount - feeCharged;
             _grantFeeToOcean(oceanId, feeCharged);
    -        IERC1155(tokenAddress).safeTransferFrom(address(this), userAddress, tokenId, amountRemaining, "");
    +        IERC1155(tokenAddress).safeTransferFrom(address(this), userAddress, tokenId, amount - feeCharged, "");
             emit Erc1155Unwrap(tokenAddress, tokenId, amount, feeCharged, userAddress, oceanId);
         }

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol

```
File: Curve2PoolAdapter.sol	

103: address tokenAddress = underlying[tokenId];

122: address tokenAddress = underlying[tokenId];
```
    diff --git a/src/adapters/Curve2PoolAdapter.sol b/src/adapters/Curve2PoolAdapter.sol
    index fce75c8..868869e 100644
    --- a/src/adapters/Curve2PoolAdapter.sol
    +++ b/src/adapters/Curve2PoolAdapter.sol
    @@ -100,10 +100,9 @@ contract Curve2PoolAdapter is OceanAdapter {
          * @param amount wrap amount
          */
         function wrapToken(uint256 tokenId, uint256 amount) internal override {
    -        address tokenAddress = underlying[tokenId];

             Interaction memory interaction = Interaction({
    -            interactionTypeAndAddress: _fetchInteractionId(tokenAddress, uint256(InteractionType.WrapErc20)),
    +            interactionTypeAndAddress: _fetchInteractionId(underlying[tokenId], uint256(InteractionType.WrapErc20)),
                 inputToken: 0,
                 outputToken: 0,
                 specifiedAmount: amount,
    @@ -119,10 +118,9 @@ contract Curve2PoolAdapter is OceanAdapter {
          * @param amount unwrap amount
          */
         function unwrapToken(uint256 tokenId, uint256 amount) internal override {
    -        address tokenAddress = underlying[tokenId];

             Interaction memory interaction = Interaction({
    -            interactionTypeAndAddress: _fetchInteractionId(tokenAddress, uint256(InteractionType.UnwrapErc20)),
    +            interactionTypeAndAddress: _fetchInteractionId(underlying[tokenId], uint256(InteractionType.UnwrapErc20)),
                 inputToken: 0,
                 outputToken: 0,
                 specifiedAmount: amount,

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol

```
File: OceanAdapter.sol	

71: uint256 unwrappedAmount = inputAmount - unwrapFee;
```
    diff --git a/src/adapters/OceanAdapter.sol b/src/adapters/OceanAdapter.sol
    index 6b3b427..ef04e22 100644
    --- a/src/adapters/OceanAdapter.sol
    +++ b/src/adapters/OceanAdapter.sol
    @@ -68,9 +68,8 @@ abstract contract OceanAdapter is IOceanPrimitive {

             // handle the unwrap fee scenario
             uint256 unwrapFee = inputAmount / IOceanInteractions(ocean).unwrapFeeDivisor();
    -        uint256 unwrappedAmount = inputAmount - unwrapFee;

    -        outputAmount = primitiveOutputAmount(inputToken, outputToken, unwrappedAmount, metadata);
    +        outputAmount = primitiveOutputAmount(inputToken, outputToken, inputAmount - unwrapFee, metadata);

             wrapToken(outputToken, outputAmount);
         }

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }

    contract Contract0 {
        uint256 num = 5;
        uint256 sum;
        function not_optimized() public returns(bool){
            uint256 num1 = num;
            sum = num1 + 2;
        }
    }

    contract Contract1 {
        uint256 num = 5;
        uint256 sum;
        function optimized() public returns(bool){
            sum = num + 2;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 63799                                     | 244             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| not_optimized                             | 24462           | 24462 | 24462  | 24462 | 1       |


| Contract1 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 63599                                     | 243             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| optimized                                 | 24460           | 24460 | 24460  | 24460 | 1       |

## [G-03] *<x> += <y>* costs more gas than *<x> = <x> + <y>* for state variables

Using the Addition operator instead of plus-equals is gas efficient.

There are 2 instances of this issue in 1 file:

```
File: Ocean.sol	

871: feeCharged += truncated;

1092: transferAmount += 1;
```

    diff --git a/src/ocean/Ocean.sol b/src/ocean/Ocean.sol
    index 1b687be..064ed8f 100644
    --- a/src/ocean/Ocean.sol
    +++ b/src/ocean/Ocean.sol
    @@ -868,7 +868,7 @@ contract Ocean is IOceanInteractions, IOceanFeeChange, OceanERC1155, IERC721Rece

                 (uint256 transferAmount, uint256 truncated) =
                     _convertDecimals(NORMALIZED_DECIMALS, decimals, amountRemaining);
    -            feeCharged += truncated;
    +            feeCharged = feeCharged + truncated;

                 _grantFeeToOcean(inputToken, feeCharged);

    @@ -1089,7 +1089,7 @@ contract Ocean is IOceanInteractions, IOceanFeeChange, OceanERC1155, IERC721Rece
                 // fully cover a potential delta, we need to transfer CEILish(amount)
                 // if truncated == 0, FLOORish(amount) == CEILish(amount)
                 // When truncated > 0, FLOORish(amount) + 1 == CEILish(AMOUNT)
    -            transferAmount += 1;
    +            transferAmount = transferAmount + 1;
                 // Now that we are transferring enough to cover the delta, we
                 // need to determine how much of the token the user is actually
                 // wrapping, in terms of 18-decimals.

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;

        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }

        function testGas() public {
            c0.add();
            c1.add1();
        }
    }

    contract Contract0 {
        uint8 num1 = 1;
        function add() public{
            num1 += 1;
        }
    }

    contract Contract1 {
        uint8 num1 = 1;
        function add1() public{
            num1 = num1 + 1;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 67017                                     | 268             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| add                                       | 5405            | 5405 | 5405   | 5405 | 1       |

| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 70623                                     | 286             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| add1                                      | 5363            | 5363 | 5363   | 5363 | 1       |

## [G-04] State variables should be cached in stack variables rather than re-reading them from storage

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

There are 2 instances of this issue in 2 files:

```
File: Curve2PoolAdapter.sol	

// @audit xToken,yToken,lpToken
209: if (((inputToken == xToken) && (outputToken == yToken)) || ((inputToken == yToken) && (outputToken == xToken)))
212: } else if (((inputToken == xToken) || (inputToken == yToken)) && (outputToken == lpTokenId)) {
214: } else if ((inputToken == lpTokenId) && ((outputToken == xToken) || (outputToken == yToken))) {        
```
    
    diff --git a/src/adapters/Curve2PoolAdapter.sol b/src/adapters/Curve2PoolAdapter.sol
    index fce75c8..be40904 100644
    --- a/src/adapters/Curve2PoolAdapter.sol
    +++ b/src/adapters/Curve2PoolAdapter.sol
    @@ -206,12 +206,15 @@ contract Curve2PoolAdapter is OceanAdapter {
             view
             returns (ComputeType computeType)
         {
    -        if (((inputToken == xToken) && (outputToken == yToken)) || ((inputToken == yToken) && (outputToken == xToken)))
    +       uint256 _xToken = xToken;
    +       uint256 _yToken = yToken;
    +       uint256 _lpTokenId = lpTokenId;
    +        if (((inputToken == _xToken) && (outputToken == _yToken)) || ((inputToken == _yToken) && (outputToken == _xToken)))
             {
                 return ComputeType.Swap;
    -        } else if (((inputToken == xToken) || (inputToken == yToken)) && (outputToken == lpTokenId)) {
    +        } else if (((inputToken == _xToken) || (inputToken == _yToken)) && (outputToken == _lpTokenId)) {
                 return ComputeType.Deposit;
    -        } else if ((inputToken == lpTokenId) && ((outputToken == xToken) || (outputToken == yToken))) {
    +        } else if ((inputToken == _lpTokenId) && ((outputToken == _xToken) || (outputToken == _yToken))) {
                 return ComputeType.Withdraw;
             } else {
                 revert INVALID_COMPUTE_TYPE();

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol

```
File: CurveTricryptoAdapter.sol	

// @audit xToken,yToken,zToken,lpToken
273: ((inputToken == xToken && outputToken == yToken) || (inputToken == yToken && outputToken == xToken))
274:     || ((inputToken == xToken && outputToken == zToken) || (inputToken == zToken && outputToken == xToken))
275:     || ((inputToken == yToken && outputToken == zToken) || (inputToken == zToken && outputToken == yToken))
279: ((inputToken == xToken) || (inputToken == yToken) || (inputToken == zToken)) && (outputToken == lpTokenId)
283: (inputToken == lpTokenId) && ((outputToken == xToken) || (outputToken == yToken) || (outputToken == zToken))
```

    diff --git a/src/adapters/CurveTricryptoAdapter.sol b/src/adapters/CurveTricryptoAdapter.sol
    index 948169f..b22f3db 100644
    --- a/src/adapters/CurveTricryptoAdapter.sol
    +++ b/src/adapters/CurveTricryptoAdapter.sol
    @@ -269,18 +269,22 @@ contract CurveTricryptoAdapter is OceanAdapter {
             view
             returns (ComputeType computeType)
         {
    +       uint256 _xToken = xToken;
    +       uint256 _yToken = yToken;
    +       uint256 _zToken = zToken;
    +       uint256 _lpTokenId = lpTokenId;
             if (
    -            ((inputToken == xToken && outputToken == yToken) || (inputToken == yToken && outputToken == xToken))
    -                || ((inputToken == xToken && outputToken == zToken) || (inputToken == zToken && outputToken == xToken))
    -                || ((inputToken == yToken && outputToken == zToken) || (inputToken == zToken && outputToken == yToken))
    +            ((inputToken == _xToken && outputToken == _yToken) || (inputToken == _yToken && outputToken == _xToken))
    +                || ((inputToken == _xToken && outputToken == _zToken) || (inputToken == _zToken && outputToken == _xToken))
    +                || ((inputToken == _yToken && outputToken == _zToken) || (inputToken == _zToken && outputToken == _yToken))
             ) {
                 return ComputeType.Swap;
             } else if (
    -            ((inputToken == xToken) || (inputToken == yToken) || (inputToken == zToken)) && (outputToken == lpTokenId)
    +            ((inputToken == _xToken) || (inputToken == _yToken) || (inputToken == _zToken)) && (outputToken == _lpTokenId)
             ) {
                 return ComputeType.Deposit;
             } else if (
    -            (inputToken == lpTokenId) && ((outputToken == xToken) || (outputToken == yToken) || (outputToken == zToken))
    +            (inputToken == _lpTokenId) && ((outputToken == _xToken) || (outputToken == _yToken) || (outputToken == _zToken))
             ) {
                 return ComputeType.Withdraw;
             } else {

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }
    
    contract Contract0 {
         uint8 num=0;
         function not_optimized() public returns(uint8){
            if(num==0)
            {
               return num;
            }
         }
    }
    
    contract Contract1 {
         uint8 num=0;
         function optimized() public returns(uint8){
            uint8 num1 = num;
            if(num1==0)
            {
               return num1;
            }
         }
    }

#### Gas Report 

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 32299                                     | 190             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| not_optimized                             | 2415            | 2415 | 2415   | 2415 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 31699                                     | 187             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| optimized                                 | 2312            | 2312 | 2312   | 2312 | 1       |

## [G-05] Instead of counting down in for statements, count up

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

There are 2 instances of this issue in 1 file:

```
File: Ocean.sol	

463: for (uint256 i = 0; i < _idLength;) {
465:     unchecked {
466:         ++i;
467:     }

501: for (uint256 i = 0; i < interactions.length;) {
537:     unchecked {
538:         ++i;
539:     }
```

    diff --git a/src/ocean/Ocean.sol b/src/ocean/Ocean.sol
    index 1b687be..c34ee89 100644
    --- a/src/ocean/Ocean.sol
    +++ b/src/ocean/Ocean.sol
    @@ -460,10 +460,10 @@ contract Ocean is IOceanInteractions, IOceanFeeChange, OceanERC1155, IERC721Rece
             BalanceDelta[] memory balanceDeltas = new BalanceDelta[](ids.length);

             uint256 _idLength = ids.length;
    -        for (uint256 i = 0; i < _idLength;) {
    +        for (uint256 i = _idLength - 1; i >= 0 ;) {
                 balanceDeltas[i] = BalanceDelta(ids[i], 0);
                 unchecked {
    -                ++i;
    +                --i;
                 }
             }

    @@ -498,7 +498,7 @@ contract Ocean is IOceanInteractions, IOceanFeeChange, OceanERC1155, IERC721Rece
                 // the locals limit and this does the trick. Is there a better way?
                 address userAddress_ = userAddress;

    -            for (uint256 i = 0; i < interactions.length;) {
    +            for (uint256 i = interactions.length - 1; i >= 0;) {
                     interaction = interactions[i];

                     (InteractionType interactionType, address externalContract) =
    @@ -535,7 +535,7 @@ contract Ocean is IOceanInteractions, IOceanFeeChange, OceanERC1155, IERC721Rece
                         balanceDeltas.increaseBalanceDelta(outputToken, outputAmount);
                     }
                     unchecked {
    -                    ++i;
    +                    --i;
                     }
                 }
             }

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.AddNum();
            c1.AddNum();
        }
    }
    
    contract Contract0 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=0;i<=9;i++){
                _num = _num +1;
            }
            num = _num;
        }
    }
    
    contract Contract1 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=9;i>=0;i--){
                _num = _num +1;
            }
            num = _num;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 77011                                     | 311             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 7040            | 7040 | 7040   | 7040 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 73811                                     | 295             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 3819            | 3819 | 3819   | 3819 | 1       |