# [G-01] Do not cache local variable used only once

[File: OceanAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L70-L75)
```
	uint256 unwrapFee = inputAmount / IOceanInteractions(ocean).unwrapFeeDivisor();
        uint256 unwrappedAmount = inputAmount - unwrapFee;

        outputAmount = primitiveOutputAmount(inputToken, outputToken, unwrappedAmount, metadata);

```

`unwrapFee`, `unwrappedAmount` are used only once, thus there's no need to declare them:

```
outputAmount = primitiveOutputAmount(inputToken, outputToken, inputAmount - (inputAmount / IOceanInteractions(ocean).unwrapFeeDivisor()), metadata);
```

[File: OceanAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L100-L102)
```
		uint256 packedValue = uint256(uint160(token));
        packedValue |= interactionType << 248;
        return bytes32(abi.encode(packedValue));
```

`packedValue` is used only once, thus there's no need to declare it:

```
 return bytes32(abi.encode(uint256(uint160(token)) | (interactionType << 248)));
```





# [G-02] There's not need to use modifier when function always revert

Function `computeInputAmount()` in `OceanAdapter.sol` is not implemented (always reverts), thus there's no need to add `onlyOcean` modifier.

[File: OceanAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L81-L94)
```
 function computeInputAmount(
        uint256 inputToken,
        uint256 outputToken,
        uint256 outputAmount,
        address userAddress,
        bytes32 maximumInputAmount
    )
        external
        override
        onlyOcean
        returns (uint256 inputAmount)
    {
        revert();
    }
```

Remove `onlyOcean` modifier:

```
 function computeInputAmount(
        uint256 inputToken,
        uint256 outputToken,
        uint256 outputAmount,
        address userAddress,
        bytes32 maximumInputAmount
    )
        external
        override
        returns (uint256 inputAmount)
    {
        revert();
    }
```


# [G-03] Calculations which won't underflow can be unchecked

[File: OceanAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L150-L156)
```
 } else if (decimalsFrom < decimalsTo) {
            // Decimal shift left (add precision)
            uint256 shift = 10 ** (uint256(decimalsTo - decimalsFrom));
            convertedAmount = amountToConvert * shift;
        } else {
            // Decimal shift right (remove precision) -> truncation
            uint256 shift = 10 ** (uint256(decimalsFrom - decimalsTo));
```

Becase we know that in `else/if` branch `decimalsFrom < decimalsTo`, we can uncheck `decimalsTo - decimalsFrom` operation.
Because we know that in `else` branch `decimalsFrom > decimalsTo`, we can uncheck `decimalsFrom - decimalsTo` operation.


The same issue occurs in `Ocean.sol` in `_convertDecimals()`:


[File: Ocean.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L1138)
```
        uint256 shift = 10 ** (uint256(decimalsTo - decimalsFrom));
        [...]
        uint256 shift = 10 ** (uint256(decimalsFrom - decimalsTo));
```


# [G-04] Address of `zToken` can be pre-calculated

[File: CurveTricryptoAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L100)
```
zToken = _calculateOceanId(address(0x4574686572), 0);
```

Function `_calculateOceanId()` returns `uint256(keccak256(abi.encodePacked(contractAddress, tokenID)))`. Since we provide it with constant values: `address(0x4574686572)` and `0`, the `zToken` can be pre-calculated.


# [G-05] Do not perform the same `if`/`else` condition twice

[File: CurveTricryptoAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L178-L235)
```
 if (action == ComputeType.Swap) {
 (...)
  } else if (action == ComputeType.Deposit) {
  (...)

  (...)

   if (action == ComputeType.Swap) {
            emit Swap(inputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);
        } else if (action == ComputeType.Deposit) {
            emit Deposit(inputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);
        } else {
            emit Withdraw(outputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);
        }
```

Function `primitiveOutputAmount()`, firstly performs `if`/`else` condition to check `action` and calculate output amount. Then, it performs the same `if`/`else` again, to emit and event. Those two `if`/`else` checks can be merged into one:

```
 if (action == ComputeType.Swap) {
            bool useEth = inputToken == zToken || outputToken == zToken;

            ICurveTricrypto(primitive).exchange{ value: inputToken == zToken ? rawInputAmount : 0 }(
                indexOfInputAmount, indexOfOutputAmount, rawInputAmount, 0, useEth
            );
        uint256 rawOutputAmount = _getBalance(underlying[outputToken]) - _balanceBefore;

        outputAmount = _convertDecimals(decimals[outputToken], NORMALIZED_DECIMALS, rawOutputAmount);

        if (uint256(minimumOutputAmount) > outputAmount) revert SLIPPAGE_LIMIT_EXCEEDED();

        emit Swap(inputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);

        (...)
```

The same issue occurs in `primitiveOutputAmount()` in `Curve2PoolAdapter.sol`


# [G-06] Check less gas costly conditions first in `if`/`else`

[File: CurveTricryptoAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L272-L288)

The first condition in `_determineComputeType()` uses multiple of AND and OR operations. Solidity has to check the whole condition (all AND/OR operators) before it proceeds to the next `else`.
It will save much more gas if we'll move `if`/`else` conditions like this:

```
 		if (
            (inputToken == lpTokenId) && ((outputToken == xToken) || (outputToken == yToken) || (outputToken == zToken))
        ) {
            return ComputeType.Withdraw;
        } else if (
            ((inputToken == xToken) || (inputToken == yToken) || (inputToken == zToken)) && (outputToken == lpTokenId)
        ) {
            return ComputeType.Deposit;
        } else if (
            ((inputToken == xToken && outputToken == yToken) || (inputToken == yToken && outputToken == xToken))
                || ((inputToken == xToken && outputToken == zToken) || (inputToken == zToken && outputToken == xToken))
                || ((inputToken == yToken && outputToken == zToken) || (inputToken == zToken && outputToken == yToken))
        ) {
            return ComputeType.Swap;
        }  else {
            revert INVALID_COMPUTE_TYPE();
        }
```

That way, when `inputToken == lpTokenId` OR when `outputToken == lpTokenId` we won't be needed to check that long OR/AND condition.



# [G-07] Use short-circuit in `if`/`else` conditions

Please notice, this is no the same as G-06. In G-06 I'm changing the order of `if`/`else` branches, while in G-07 I'm changing the order of the condition itself.

Solidity uses short-circut when evaluating conditions. E.g. in `f(x) AND g(x)` - when `f(x)` is evaluated as false, `g(x)` won't be called (because false AND whatever is false).
The same for OR: `f(x) OR g(x)` - when `f(x)` is evaluated as true, `g(x)` won't be called (because true OR whatever is true).

This allows to write condition in more optimized way.

[File: Curve2PoolAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L212)
```
 } else if (((inputToken == xToken) || (inputToken == yToken)) && (outputToken == lpTokenId)) {
```

can be rewritten to:

```
 } else if ((outputToken == lpTokenId) && ((inputToken == xToken) || (inputToken == yToken))) {
```

This will lead us to below results:
`(outputToken == lpTokenId)` uses less gas than `((inputToken == xToken) || (inputToken == yToken))`  (one `==` vs `== OR ==`).
 When `(outputToken == lpTokenId)` evaluates to `false`, because of the `AND` operator, there won't be any need to calculate more gas-costly `((inputToken == xToken) || (inputToken == yToken))`.


[File: CurveTricryptoAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L279)
```
((inputToken == xToken) || (inputToken == yToken) || (inputToken == zToken)) && (outputToken == lpTokenId)
```

can be rewritten to:

```
(outputToken == lpTokenId) && ((inputToken == xToken) || (inputToken == yToken) || (inputToken == zToken))
```

This will lead us to below results:
`(outputToken == lpTokenId)` uses less gass than `((inputToken == xToken) || (inputToken == yToken) || (inputToken == zToken))` (one `==` vs `== OR == OR == `).

When `(outputToken == lpTokenId)` evaluates to `false`, because of the `AND` operator, there won't be any need to calculate more gas-costly: `((inputToken == xToken) || (inputToken == yToken) || (inputToken == zToken))`.


[File: Ocean.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L1043)
```
 if (_isNotTokenOfPrimitive(outputToken, primitive) && (outputAmount > 0)) {
```

`(outputAmount > 0)` costs less gas than `_isNotTokenOfPrimitive(outputToken, primitive)`, so it's better to change the order:
```
 if ((outputAmount > 0) && _isNotTokenOfPrimitive(outputToken, primitive)) {
```


# [G-08] `unwrappedAmount` calculation can be unchecked

[File: OceanAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L70-L71)
```
        uint256 unwrapFee = inputAmount / IOceanInteractions(ocean).unwrapFeeDivisor();
        uint256 unwrappedAmount = inputAmount - unwrapFee;
```

Since `unwrapFee = inputAmount / IOceanInteractions(ocean).unwrapFeeDivisor()`, we know, that `inputAmount >= unwrapFee`, thus `uint256 unwrappedAmount = inputAmount - unwrapFee;` won't underflow and can be unchecked.


# [G-09] Cache `msg.value` instead of using it multiple of times

Assing `msg.value` to local variable and use that variable, instead of using `msg.value` every time.

[File: Ocean.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L391-L396)
```
 if (msg.value != 0) {
            inputToken = 0;
            inputAmount = 0;
            outputToken = WRAPPED_ETHER_ID;
            outputAmount = msg.value;
            emit EtherWrap(msg.value, userAddress);
```


# [G-10] `specifiedAmount` can be hardcoded for ERC-721

[File: Ocean.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L640-L653)
```
 } else if (interactionType == InteractionType.WrapErc721) {
            // An ERC-20 or ERC-1155 contract can have a transfer with
            // any amount including zero. Here, we need to require that
            // the specifiedAmount is equal to one, since the external
            // call to the ERC-721 contract does not include an amount,
            // and the ledger is mutated based on the specifiedAmount.
            if (specifiedAmount != 1) revert INVALID_ERC721_AMOUNT();
            inputToken = 0;
            inputAmount = 0;
            outputToken = specifiedToken;
            outputAmount = specifiedAmount;
            _erc721Wrap(externalContract, uint256(interaction.metadata), userAddress, outputToken);
        } else if (interactionType == InteractionType.UnwrapErc721) {
            // See the comment in the preceeding else if block.
            if (specifiedAmount != 1) revert INVALID_ERC721_AMOUNT();
            inputToken = specifiedToken;
            inputAmount = specifiedAmount;
            outputToken = 0;
            outputAmount = 0;
            _erc721Unwrap(externalContract, uint256(interaction.metadata), userAddress, inputToken);
```

Whenever we wrapping or unwrapping ERC-721, `specifiedAmount` has always be `1`. Otherwise function will revert with `INVALID_ERC721_AMOUNT()`. This implies, that we can save one read of `outputAmount` and use `1` straightforward:

```
 } else if (interactionType == InteractionType.WrapErc721) {
            // An ERC-20 or ERC-1155 contract can have a transfer with
            // any amount including zero. Here, we need to require that
            // the specifiedAmount is equal to one, since the external
            // call to the ERC-721 contract does not include an amount,
            // and the ledger is mutated based on the specifiedAmount.
            if (specifiedAmount != 1) revert INVALID_ERC721_AMOUNT();
            inputToken = 0;
            inputAmount = 0;
            outputToken = specifiedToken;
            outputAmount = 1;
            _erc721Wrap(externalContract, uint256(interaction.metadata), userAddress, outputToken);
        } else if (interactionType == InteractionType.UnwrapErc721) {
            // See the comment in the preceeding else if block.
            if (specifiedAmount != 1) revert INVALID_ERC721_AMOUNT();
            inputToken = specifiedToken;
            inputAmount = 1;
            outputToken = 0;
            outputAmount = 0;
            _erc721Unwrap(externalContract, uint256(interaction.metadata), userAddress, inputToken);
```


# [G-11] Do not declare variables inside loop

[File: Ocean.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L501-L541)
```
for (uint256 i = 0; i < interactions.length;) {
     [...]
    (InteractionType interactionType, address externalContract) =
        _unpackInteractionTypeAndAddress(interaction);

    [...]
    uint256 specifiedToken = _getSpecifiedToken(interactionType, externalContract, interaction);
    [...]
    uint256 specifiedAmount;
    [...]
     (uint256 inputToken, uint256 inputAmount, uint256 outputToken, uint256 outputAmount) =
```

Declare variable outside loop:
```
InteractionType interactionType;
address externalContract;
uint256 specifiedToken;
uint256 specifiedToken;
uint256 specifiedAmount;
uint256 inputToken;
uint256 inputAmount;
uint256 outputToken;
uint256 outputAmount;
for (uint256 i = 0; i < interactions.length;) {
     [...]
    (interactionType, externalContract) =
        _unpackInteractionTypeAndAddress(interaction);

    [...]
    specifiedToken = _getSpecifiedToken(interactionType, externalContract, interaction);
    [...]
    specifiedAmount;
    [...]
     (inputToken, inputAmount, outputToken, outputAmount) =
```

That way, we won't use gas on variable declaration on every loop iteration.


# [G-12] Approve tokens directly in `constructor`, instead of calling `_approveToken()`

This issue affects both `CurveTricryptoAdapter.sol` and `Curve2PoolAdapter.sol`.
For the report clarity, I'll describe how it works for `Curve2PoolAdapter.sol` only. The same recommendation should be implemented in `CurveTricryptoAdapter.sol` too.

Function `_approveToken()` is defined as below:

[File: Curve2PoolAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L189-L192)
```
 function _approveToken(address tokenAddress) private {
        IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);
        IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);
    }

```

As we see, on every call, it uses gas on reading `ocean` and `privimitive` variables.

This function is utilized in the constructor:

[File: Curve2PoolAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L77C-L95)
```
 constructor(address ocean_, address primitive_) OceanAdapter(ocean_, primitive_) {
        address xTokenAddress = ICurve2Pool(primitive).coins(0);
        [...]
        _approveToken(xTokenAddress);

        address yTokenAddress = ICurve2Pool(primitive).coins(1);
        [...]
        _approveToken(yTokenAddress);

        [...]
        _approveToken(primitive_);
    }
```

This suggests, that it reads `ocean` and `primitive` variables variables 3 times (because it's called three times). This is extremely ineffective. Much better idea would be to cache `ocean` and `primitive` into local variables and use `IERC20Metadata(tokenAddress).approve` directly. We'll reduce the number of `ocean` and `primitive` reads.