# QA Report

## Summary

| id | title |
| --- | --- |
| [L-01](#l-01-curvetricryptoadapter-assumes-that-weth-is-always-the-last-token-in-the-pool) | `CurveTricryptoAdapter` assumes that `WETH` is always the last token in the pool |
| [L-01](#l-02-dust-from-18-decimal-tokens-will-collect-in-curve-pool-adapters)| Dust from >18 decimal tokens will collect in Curve pool adapters |
| [NC-01](#nc-01-erroneous-commment) | Erroneous commment |
| [NC-02](#nc-02-inconsistent-use-of-primitiveprimitive_-in-curve2pooladapter-constructor) | Inconsistent use of `primitive`/`primitive_` in `Curve2PoolAdapter` constructor |
| [NC-03](#nc-03-variables-not-used-in-every-branch) | Variables not used in every branch |

## Low

### L-01 `CurveTricryptoAdapter` assumes that `WETH` is always the last token in the pool

When deploying a `CurveTricryptoAdapter` each token is queried and their decimals are stored in the contract.

One potential problem here is that the code assumes that `WETH` is the last token in the Curve tricrypto pool:

[`CurveTricryptoAdapter::constructor`](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L99-L100):
```solidity
File: src/adapters/CurveTricryptoAdapter.sol

 99:        address wethAddress = ICurveTricrypto(primitive).coins(2);
100:        zToken = _calculateOceanId(address(0x4574686572), 0); // hexadecimal(ascii("Ether"))
```

This must not be the case. On mainnet there are plenty of examples of tricrypto pools where `WETH` is not the last token. However, this protocol is only going to be deployed on Arbitrum, where the only two tricrypto pools _does_ have `WETH` as the last token. Hence it will work. Just mind that if a new tricrypto pool is deployed on Arbitrum, there's no guarantee that it will have `WETH` as the last token.

If `WETH` weren't the last token the `CurveTricryptoAdapter` would behave very weirdly together with `Ocean` as the assumption of which tokens are which would not work. This would at best cause the `CurveTricryptoAdapter` not to work, at worst it could mess up which tokens are moved and how many decimals they have.

Further reading:
https://solodit.xyz/issues/h-2-curvetricryptooracle-incorrectly-assumes-that-weth-is-always-the-last-token-in-the-pool-which-leads-to-bad-lp-pricing-sherlock-none-blueberry-update-3-git

#### Recommendation
Consider making which position is `WETH` dynamic.

### L-02 Dust from >18 decimal tokens will collect in Curve pool adapters
When swapping in Curve pools the amounts are converted back and forth between whatever the token decimal is and Oceans 18 decimal accounting. The issue is that if the token converted has more than 18 decimals the left overs are truncated:

[`Curve2PoolAdapter::primitiveOutputAmount`](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L173):
```solidity
File: src/adapters/Curve2PoolAdapter.sol

173:        outputAmount = _convertDecimals(decimals[outputToken], NORMALIZED_DECIMALS, rawOutputAmount);
```

If `decimals[outputToken]` is greater than `NORMALIZED_DECIMALS`, that will cause truncation in [`OceanAdapter::_convertDecimals`](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L154-L158):
```solidity
File: src/adapters/OceanAdapter.sol

154:        } else {
155:            // Decimal shift right (remove precision) -> truncation
156:            uint256 shift = 10 ** (uint256(decimalsFrom - decimalsTo));
157:            convertedAmount = amountToConvert / shift;
158:        }
```

Then the amount is, again, scaled up in [`Ocean::_convertDecimals`](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L1136-L1141):
```solidity
File: src/ocean/Ocean.sol

1136:        } else if (decimalsFrom < decimalsTo) {
1137:            // Decimal shift left (add precision)
1138:            uint256 shift = 10 ** (uint256(decimalsTo - decimalsFrom));
1139:            convertedAmount = amountToConvert * shift;
1140:            truncatedAmount = 0;
1141:        } else {
```

Imagine a 24 decimal token. The swap resultus in an amount `123_456_789_012_345_678_901_234`. This will be converted to `Ocean` 18 decimals, `123_456_789_012_345_678` in Ocean accounting. Then when the wrapping of the ERC20 happens, it is scaled up to 24 decimals again. Here, however without the last information: `123_456_789_012_345_678_000_000`. Thus, `901_234` of the token will be left in the Curve pool adapter contract.

#### Recommendation
Consider adding a public sweep option to the Curve pool adapters, could be locked to owner or public.

## Non critical / Informational

### NC-01 Erroneous commment

[`Ocean.sol#L83-L84`](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L83-L84)
```solidity
File: src/ocean/Ocean.sol

83:    /// @dev hexadecimal(ascii("shETH"))
84:    uint256 public immutable WRAPPED_ETHER_ID;
```

While in the constructor, the value set is actually `hexadecimal(ascii("Ether"))`:

[`Ocean::constructor`](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L173)
```solidity
File: src/ocean/Ocean.sol

173:        WRAPPED_ETHER_ID = _calculateOceanId(address(0x4574686572), 0); // hexadecimal(ascii("Ether"))
```

### NC-02 Inconsistent use of `primitive`/`primitive_` in `Curve2PoolAdapter` constructor

There's a mixed usage of the immutable field `primitive` and the passed calldata parameter `primitive_` in constructor of [`Curve2PoolAdapter`](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L77-L95):
```solidity
File: src/adapters/Curve2PoolAdapter.sol

77:    constructor(address ocean_, address primitive_) OceanAdapter(ocean_, primitive_) {
           // `primitive` (no underscore) used here
78:        address xTokenAddress = ICurve2Pool(primitive).coins(0);
...
           // `primitive` (no underscore) used here
84:        address yTokenAddress = ICurve2Pool(primitive).coins(1);
...
           // `primitive_` (underscore) used here
91:        lpTokenId = _calculateOceanId(primitive_, 0);
...
95:    }
```

Consider just using one of them, `primitive` (no underscore) seems the more used one, since it's also used in the constructor of `CurveTricryptoAdapter`

### NC-03 Variables not used in every branch

In `primitiveOutputAmount` for both `Curve2PoolAdapter` and `CurveTricryptoAdapter` the two variables `indexOfInputAmount` and `indexOfOutputAmount` are loaded before they are used:

[`Curve2PoolAdapter::primitiveOutputAmount`](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L159-L160):
```solidity
File: src/adapters/Curve2PoolAdapter.sol

159:        int128 indexOfInputAmount = indexOf[inputToken];
160:        int128 indexOfOutputAmount = indexOf[outputToken];
```

However, both are not used in any of the following branches:
[`Curve2PoolAdapter::primitiveOutputAmount`](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L162-L171):
```solidity

162:        if (action == ComputeType.Swap) {
163:            rawOutputAmount =
164:                ICurve2Pool(primitive).exchange(indexOfInputAmount, indexOfOutputAmount, rawInputAmount, 0);
165:        } else if (action == ComputeType.Deposit) {
166:            uint256[2] memory inputAmounts;
167:            inputAmounts[uint256(int256(indexOfInputAmount))] = rawInputAmount;
168:            rawOutputAmount = ICurve2Pool(primitive).add_liquidity(inputAmounts, 0);
169:        } else {
170:            rawOutputAmount = ICurve2Pool(primitive).remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);
171:        }
```

Here you can see, different combinations of `indexOfInputAmount` and `indexOfOutputAmount` are used in different branches. But declaring them where they are declaed hints that both would be used in all branches which is not the case. Loading them like this also increases gas cost for branches where only one of them is used.

Consider reading and declaring only the necessary field when it is used.