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
When unwrapping an ERC-20 token whose `decimals()` is greater than 18, the portion beyond the 18th decimal place is discarded outright. For instance, the amount for a token with 21 decimals will be unwrapped as 123.123456789012345678 when the actual amount could have been 123.123456789012345678901, leaving/truncating 0.000000000000000000901.

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L864-L880

```solidity
    function _erc20Unwrap(address tokenAddress, uint256 amount, address userAddress, uint256 inputToken) private {
        try IERC20Metadata(tokenAddress).decimals() returns (uint8 decimals) {
            uint256 feeCharged = _calculateUnwrapFee(amount);
            uint256 amountRemaining = amount - feeCharged;

            (uint256 transferAmount, uint256 truncated) =
                _convertDecimals(NORMALIZED_DECIMALS, decimals, amountRemaining);
            feeCharged += truncated;

            _grantFeeToOcean(inputToken, feeCharged);

            SafeERC20.safeTransfer(IERC20(tokenAddress), userAddress, transferAmount);
            emit Erc20Unwrap(tokenAddress, transferAmount, amount, feeCharged, userAddress, inputToken);
        } catch {
            revert NO_DECIMAL_METHOD();
        }
    }
```
I recommend saving/adding the truncated amount to the caller since the the DAO's balance of a token in an 18 decimal representation won't be getting it.  

## [L-04] USDT might turn into a fee-on-transfer token
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

## [L-05] Function arguments not efficiently used
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
## [L-06] Ineffective CEI adapted from OpenZeppelin Reentrancy Guard
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
## [L-07] Private function with embedded modifier reduces contract size
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