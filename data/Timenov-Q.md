## Summary
[NC-1] Enum is defined at the wrong place.
[NC-2] Mappings definition is confusing.
[NC-3] Space left in comments.
[NC-4] No need to assing value to 0.
[NC-5] `else` logic can be refactored.
[NC-6] Unsafe call to `decimals()`.
[NC-7] Mapping not updated.
[NC-8] Consider adding `minAmountOut` parameter.

### [NC-1] Enum is defined at the wrong place.
The enum `ComputeType` is defined outside of the contract. It should be defined inside the contract definition.

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L10-L14

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L15-L19

### [NC-2] Mappings definition is confusing.
In `Curve2PoolAdapter.sol` we have 2 mappings: `indexOf` and `decimals`. They both have `@notice`, however it is not in the same format.

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L64-L68

The `@notice` for `indexOf` is:

```solidity
@notice map token Ocean IDs to corresponding Curve pool indices
```

The `@notice` for `decimals` is:

```solidity
@notice The underlying token decimals wrt to the Ocean ID
```

In the first mapping we use the format: `maps key to value` and in the second we use `value mapped to key`. This is a bit confusing, so consider using the same format.

### [NC-3] Space left in comments.
Empty spaces are left in some comments. Consider removing them.

```solidity
            |    
require the  same level of trust as direct balance transfers.
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L183

```solidity
                              |
Ether payments are push only.  We always wrap ERC-X tokens using pull
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L387

### [NC-4] No need to assing value to 0.
There are 3 instances of this issue.

This is the first one. `dust` is named export variable and has 0 by default. The whole `else` block can be removed.

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L1103-L1108

These 2 we can just remove `truncatedAmount = 0;` because it is a named export variable and has 0 by default.

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L1135

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L1140

### [NC-5] `else` logic can be refactored.
In `Ocean.sol` there are 2 `else` blocks that have `assert` statement in them. This does not look good and we can change the logic a bit to make it look better. For example change this:

```solidity
        } else {
            assert(interactionType == InteractionType.UnwrapEther && specifiedToken == WRAPPED_ETHER_ID);
            inputToken = specifiedToken;
            inputAmount = specifiedAmount;
            outputToken = 0;
            outputAmount = 0;
            _etherUnwrap(inputAmount, userAddress);
        }
```

To this:

```solidity
        } else if (interactionType == InteractionType.UnwrapEther && specifiedToken == WRAPPED_ETHER_ID) {
            inputToken = specifiedToken;
            inputAmount = specifiedAmount;
            outputToken = 0;
            outputAmount = 0;
            _etherUnwrap(inputAmount, userAddress);
        } else {
            revert();
        }
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L666-L673

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L720-L723

### [NC-6] Unsafe call to `decimals()`.
The `decimals` function is optional in the initial ERC20 and might fail for old tokens that do not implement it.

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L81

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L88

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L93

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L89

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L96

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L109

In `Ocean::_erc20Wrap` try/catch is used for the `decimals()`. Consider using it here as well, also this solution might be helpful: https://github.com/boringcrypto/BoringSolidity/blob/c73ed73afa9273fbce93095ef177513191782254/contracts/libraries/BoringERC20.sol#L49-L55

### [NC-7] Mapping not updated.
In `Curve2PoolAdapter.sol`'s constructor we update the `indexOf` mapping of `yToken`, but we do not update for `xToken`.

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L86

### [NC-8] Consider adding `minAmountOut` parameter.
Consider adding `minAmountOut` parameter in `computeOutputAmount` in `OceanAdapter`. This will allow users to pass value to this parameter so they receive no less tokens amount than the parameter's value. If the value passed is 0, this will mean that the user does not care how much tokens he will receive. Also add check `require(outputAmount >= minAmountOut)`.

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L55-L76