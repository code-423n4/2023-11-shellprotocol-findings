## [L-01] `Ocean._calculateUnwrapFee()` should be rounded up
`feeCharged` should adopt the `ceil` instead of floor behavior of integer division in favor of updating the DAO's balance.

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L1158-L1160

```solidity
    function _calculateUnwrapFee(uint256 unwrapAmount) private view returns (uint256 feeCharged) {
        feeCharged = unwrapAmount / unwrapFeeDivisor;
    }  
```
I recommend adopting a math library to better handle the rounding matter.

## [L-02] Unnecessary zero dust placing on the owner's balance
In `Ocean._erc20Wrap()`, `dust` returned by `_determineTransferAmount()` could be 0. In fact, `dust` typically/mostly equals 0, e.g. a 168 USDC amount of the ERC-20 token to be wrapped in terms of 18-decimal fixed point would be like 168_000000_000000000000 where `dust == 0`.

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L1103-L1108

```solidity
        } else {
            // if truncated == 0, then we don't need to do anything fancy to
            // determine transferAmount, the result _convertDecimals() returns
            // is correct.
            dust = 0;
        }
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L827-L834

```solidity
            (transferAmount, dust) = _determineTransferAmount(amount, decimals);

            // If the user is unwrapping a delta, the residual dust could be
            // written to the user's ledger balance. However, it costs the
            // same amount of gas to place the dust on the owner's balance,
            // and accumulation of dust may eventually result in
            // transferrable units again.
            _grantFeeToOcean(outputToken, dust);
```
I recommend having the affected code line refactored as follows:

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L834

```diff
-            _grantFeeToOcean(outputToken, dust);
+            if (dust != 0) { 
+                _grantFeeToOcean(outputToken, dust);
+            }
``` 
## [L-03] Dust associated with token decimals over 18 not catered for
When dealing with an ERC-20 token whose `decimals()` is greater than 18, the portion beyond the 18th decimal place is discarded outright. For instance, when `rawOutputAmount` returned by `Curve2PoolAdapter.primitiveOutputAmount()` for a token with 21 decimals was 123.123456789012345678901, `_convertDecimals()` would be invoked to assign `outputAmount` as `123.123456789012345678` to be wrapped via [wrapToken()](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L102-L114), leaving/truncating 0.000000000000000000901.

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L162-L173

```solidity
        if (action == ComputeType.Swap) {
            rawOutputAmount =
                ICurve2Pool(primitive).exchange(indexOfInputAmount, indexOfOutputAmount, rawInputAmount, 0);
        } else if (action == ComputeType.Deposit) {
            uint256[2] memory inputAmounts;
            inputAmounts[uint256(int256(indexOfInputAmount))] = rawInputAmount;
            rawOutputAmount = ICurve2Pool(primitive).add_liquidity(inputAmounts, 0);
        } else {
            rawOutputAmount = ICurve2Pool(primitive).remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);
        }

        outputAmount = _convertDecimals(decimals[outputToken], NORMALIZED_DECIMALS, rawOutputAmount);
```
I recommend adding a function that could be used to withdraw the accumulated truncated amount by the owner when it has become sizable enough to be transferable rather than leaving it stuck forever in the contract.  

## [L-04] Slippage could have been detected earlier
`Curve2PoolAdapter.primitiveOutputAmount()` and `CurveTricryptoAdapter.primitiveOutputAmount()` both have `0` slippage adopted by default when doing deposit, swap or withdraw, and doing the slippage check only after the `rawOutputAmount` has been converted to the 18 decimal format.  

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L162-L175

```solidity
        if (action == ComputeType.Swap) {
            rawOutputAmount =
                ICurve2Pool(primitive).exchange(indexOfInputAmount, indexOfOutputAmount, rawInputAmount, 0);
        } else if (action == ComputeType.Deposit) {
            uint256[2] memory inputAmounts;
            inputAmounts[uint256(int256(indexOfInputAmount))] = rawInputAmount;
            rawOutputAmount = ICurve2Pool(primitive).add_liquidity(inputAmounts, 0);
        } else {
            rawOutputAmount = ICurve2Pool(primitive).remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);
        }

        outputAmount = _convertDecimals(decimals[outputToken], NORMALIZED_DECIMALS, rawOutputAmount);

        if (uint256(minimumOutputAmount) > outputAmount) revert SLIPPAGE_LIMIT_EXCEEDED();
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L198-L227

```solidity
        if (action == ComputeType.Swap) {
            bool useEth = inputToken == zToken || outputToken == zToken;

            ICurveTricrypto(primitive).exchange{ value: inputToken == zToken ? rawInputAmount : 0 }(
                indexOfInputAmount, indexOfOutputAmount, rawInputAmount, 0, useEth
            );
        } else if (action == ComputeType.Deposit) {
            uint256[3] memory inputAmounts;
            inputAmounts[indexOfInputAmount] = rawInputAmount;

            if (inputToken == zToken) IWETH(underlying[zToken]).deposit{ value: rawInputAmount }();

            ICurveTricrypto(primitive).add_liquidity(inputAmounts, 0);
        } else {
            if (outputToken == zToken) {
                uint256 wethBalance = IERC20Metadata(underlying[zToken]).balanceOf(address(this));
                ICurveTricrypto(primitive).remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);
                IWETH(underlying[zToken]).withdraw(
                    IERC20Metadata(underlying[zToken]).balanceOf(address(this)) - wethBalance
                );
            } else {
                ICurveTricrypto(primitive).remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);
            }
        }

        uint256 rawOutputAmount = _getBalance(underlying[outputToken]) - _balanceBefore;

        outputAmount = _convertDecimals(decimals[outputToken], NORMALIZED_DECIMALS, rawOutputAmount);

        if (uint256(minimumOutputAmount) > outputAmount) revert SLIPPAGE_LIMIT_EXCEEDED();
```
I recommend converting `minimumOutputAmount` to the supposed token decimal format (just like it has been done so on `inputAmount`) via `_convertDecimals()` (round it up if need be), and then using the converted slippage amount instead of `0` when calling `exchange()`, `add_liquidity()` or `remove_liquidity_one_coin()`.  

## [L-05] USDT might turn into a fee-on-transfer token
According to the contract NatSpec of Curve2PoolAdapter.sol, the adapter is enabling deposit, swap, and withdraw on the USDC-USDT pair on curve: 

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L16-L19

```solidity
/**
 * @notice
 *   curve2pool adapter contract enabling swapping, adding liquidity & removing liquidity for the curve usdc-usdt pool
 */
``` 
Nevertheless, USDT can activate fees and become a fee-on-transfer token, breaking all associated primitive interactions.

I recommend planning for this contingency by having a graceful primitive exit/cease route from the Ocean. 

## [L-06] Function arguments not efficiently used
When deploying Curve2PoolAdapter.sol and CurveTricryptoAdapter.sol, the inputted parameter `primitive_` could have been fully utilized instead of resorting to the immutable `primitive`.

I recommend refactoring the affected codes as follows:

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L77-L95

```diff
    constructor(address ocean_, address primitive_) OceanAdapter(ocean_, primitive_) {
-        address xTokenAddress = ICurve2Pool(primitive).coins(0);
+        address xTokenAddress = ICurve2Pool(primitive_).coins(0);
        xToken = _calculateOceanId(xTokenAddress, 0);
        underlying[xToken] = xTokenAddress;
        decimals[xToken] = IERC20Metadata(xTokenAddress).decimals();
        _approveToken(xTokenAddress);

-        address yTokenAddress = ICurve2Pool(primitive).coins(1);
+        address yTokenAddress = ICurve2Pool(primitive_).coins(1);
        yToken = _calculateOceanId(yTokenAddress, 0);
        indexOf[yToken] = int128(1);
        underlying[yToken] = yTokenAddress;
        decimals[yToken] = IERC20Metadata(yTokenAddress).decimals();
        _approveToken(yTokenAddress);

        lpTokenId = _calculateOceanId(primitive_, 0);
        underlying[lpTokenId] = primitive_;
        decimals[lpTokenId] = IERC20Metadata(primitive_).decimals();
        _approveToken(primitive_);
    }
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L85-L111

```diff
    constructor(address ocean_, address primitive_) OceanAdapter(ocean_, primitive_) {
-        address xTokenAddress = ICurveTricrypto(primitive).coins(0);
+        address xTokenAddress = ICurveTricrypto(primitive_).coins(0);
        xToken = _calculateOceanId(xTokenAddress, 0);
        underlying[xToken] = xTokenAddress;
        decimals[xToken] = IERC20Metadata(xTokenAddress).decimals();
        _approveToken(xTokenAddress);

-        address yTokenAddress = ICurveTricrypto(primitive).coins(1);
-        address yTokenAddress = ICurveTricrypto(primitive_).coins(1);
        yToken = _calculateOceanId(yTokenAddress, 0);
        indexOf[yToken] = 1;
        underlying[yToken] = yTokenAddress;
        decimals[yToken] = IERC20Metadata(yTokenAddress).decimals();
        _approveToken(yTokenAddress);

-        address wethAddress = ICurveTricrypto(primitive).coins(2);
+        address wethAddress = ICurveTricrypto(primitive_).coins(2);
        zToken = _calculateOceanId(address(0x4574686572), 0); // hexadecimal(ascii("Ether"))
        indexOf[zToken] = 2;
        underlying[zToken] = wethAddress;
        decimals[zToken] = NORMALIZED_DECIMALS;
        _approveToken(wethAddress);

-        address lpTokenAddress = ICurveTricrypto(primitive).token();
+        address lpTokenAddress = ICurveTricrypto(primitive_).token();
        lpTokenId = _calculateOceanId(lpTokenAddress, 0);
        underlying[lpTokenId] = lpTokenAddress;
        decimals[lpTokenId] = IERC20Metadata(lpTokenAddress).decimals();
        _approveToken(lpTokenAddress);
    }
```
## [L-07] Ineffective CEI adapted from OpenZeppelin Reentrancy Guard
In Ocean.sol, sandwiching the function logic of `_erc721Wrap()` and `_erc1155Wrap()` with `NOT_INTERACTION` and `INTERACTION` is essentially not doing much as far as reentrancy guard is concerned in the absence of any conditional or modifier check(s):

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L889-L894

```solidity
    function _erc721Wrap(address tokenAddress, uint256 tokenId, address userAddress, uint256 oceanId) private {
        _ERC721InteractionStatus = INTERACTION;
        IERC721(tokenAddress).safeTransferFrom(userAddress, address(this), tokenId);
        _ERC721InteractionStatus = NOT_INTERACTION;
        emit Erc721Wrap(tokenAddress, tokenId, userAddress, oceanId);
    }
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L920-L934

```solidity
    function _erc1155Wrap(
        address tokenAddress,
        uint256 tokenId,
        uint256 amount,
        address userAddress,
        uint256 oceanId
    )
        private
    {
        if (tokenAddress == address(this)) revert NO_RECURSIVE_WRAPS();
        _ERC1155InteractionStatus = INTERACTION;
        IERC1155(tokenAddress).safeTransferFrom(userAddress, address(this), tokenId, amount, "");
        _ERC1155InteractionStatus = NOT_INTERACTION;
        emit Erc1155Wrap(tokenAddress, tokenId, amount, userAddress, oceanId);
    }  
```
## [L-08] Private function with embedded modifier reduces contract size
Consider having the logic of a modifier embedded through a private function to reduce contract size if need be. A `private` visibility that saves more gas on function calls than the `internal` visibility is adopted because the modifier will only be making this call inside the contract.

For instance, the modifier inside the big Ocean contract may be refactored as follows:

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L185-L188

```diff
+    function _onlyApprovedForwarder(address userAddress) private view {
+        if (!isApprovedForAll(userAddress, msg.sender)) revert FORWARDER_NOT_APPROVED();
+    }

      modifier onlyApprovedForwarder(address userAddress) {
-        if (!isApprovedForAll(userAddress, msg.sender)) revert FORWARDER_NOT_APPROVED();
+        _onlyApprovedForwarder(address userAddress);
      _;
      }
```
## [L-09] Optional `deadline` parameter for `primitiveOutputAmount()`
This is unlikely to happen on Arbitrum or any other L2 chain, but, should the protocol plan on making the Ocean-Primitive interactions on the Ethereum mainnet, a `deadline` option could further protect callers from changes beyond the mitigation capability offered by the slippage. For instance, swapping USDC for USDT a day later would be getting a higher `rawOutputAmount`. 

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L162-L164

```solidity
        if (action == ComputeType.Swap) {
            rawOutputAmount =
                ICurve2Pool(primitive).exchange(indexOfInputAmount, indexOfOutputAmount, rawInputAmount, 0);
```
However, due to congestion and bot MEV re-ordering, a deFi call transacted a day later would not have an expectedly higher `rawOutputAmount` received simply because a higher `minimumOutputAmount` had not been entered for better slippage protection.  

## [L-10] Excess contract balance not catered for
Ideally, a well-designed smart contract should not retain excess funds inadvertently. Any excess balance remaining after operations should be accounted for and handled appropriately, either by returning to the user, contributing to the next operation, or managed through specific functions designed for balance withdrawal or reallocation.

For instance, in CurveTricryptoAdapter.sol, ETH could be sent in accidentally via [fallback()](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L291). This could render the excess ETH stuck in the contract forever because `primitiveOutputAmount()` handles a balance check that calculates the exact amount of tokens received in the Curve operation. The same shall apply to other token types inadvertently received by the contract. 

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L223

```solidity
        uint256 rawOutputAmount = _getBalance(underlying[outputToken]) - _balanceBefore;
```
I recommend having a function the owner can make withdraws on the stuck excess funds when needed.

## [NC-01] Incorrect comments
`Ocean.forwardedDoInteraction()` is making call to `_doInteraction()` instead of `_doMultipleInteractions()`.

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L256-L268

```solidity
    function forwardedDoInteraction(
        Interaction calldata interaction,
        address userAddress
    )
        external
        payable
        override
        onlyApprovedForwarder(userAddress)
        returns (uint256 burnId, uint256 burnAmount, uint256 mintId, uint256 mintAmount)
    {
        emit ForwardedOceanTransaction(msg.sender, userAddress, 1);
        return _doInteraction(interaction, userAddress);
    }
```
As such, I recommend having the comments corrected as follows to avoid confusing readers:

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L252

```diff
-     * @dev call to _doMultipleInteractions() forwards the userAddress
+     * @dev call to _doInteractions() forwards the userAddress
```
## [NC-02] Typo errors
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L49

```diff
- *  external contracts at certain points in its exection. The lifecycle is
+ *  external contracts at certain points in its execution. The lifecycle is
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L756

```diff
-        // mint before making a external call to the primitive to integrate with external protocol primitive adapters
+        // mint before making an external call to the primitive to integrate with external protocol primitive adapters
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L797

```diff
-        // burn before making a external call to the primitive to integrate with external protocol primitive adapters
+        // burn before making a external call to the primitive to integrate with external protocol primitive adapters
```
## [NC-03] Activate the optimizer
Before deploying your contract, activate the optimizer when compiling using “solc --optimize --bin sourceFile.sol”. By default, the optimizer will optimize the contract assuming it is called 200 times across its lifetime. If you want the initial contract deployment to be cheaper and the later function executions to be more expensive, set it to “ --optimize-runs=1”. Conversely, if you expect many transactions and do not care for higher deployment cost and output size, set “--optimize-runs” to a high number.

```
module.exports = {
solidity: {
version: "0.8.20",
settings: {
  optimizer: {
    enabled: true,
    runs: 1000,
  },
},
},
};
```
Please visit the following site for further information:

https://docs.soliditylang.org/en/v0.5.4/using-the-compiler.html#using-the-commandline-compiler

Here's one example of instance on opcode comparison that delineates the gas saving mechanism:

```
for !=0 before optimization
PUSH1 0x00
DUP2
EQ
ISZERO
PUSH1 [cont offset]
JUMPI

after optimization
DUP1
PUSH1 [revert offset]
JUMPI
```
Disclaimer: There have been several bugs with security implications related to optimizations. For this reason, Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them. Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past . A high-severity bug in the emscripten -generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG. Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. Please measure the gas savings from optimizations, and carefully weigh them against the possibility of an optimization-related bug. Also, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.