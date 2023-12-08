## [L-01] Incorrect comment.
1. The comment for function `CurveTricryptoAdapter._determineComputeType` is incorrect. It is copied from the comment for function `Curve2PoolAdapter._determineComputeType`.
```solidity
257:    /**
258:     * @dev Uses the inputToken and outputToken to determine the ComputeType
259:     *  (input: xToken, output: yToken) | (input: yToken, output: xToken) => SWAP
260:     *  base := xToken | yToken
261:     *  (input: base, output: lpToken) => DEPOSIT
262:     *  (input: lpToken, output: base) => WITHDRAW
263:     */
264:    function _determineComputeType(
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L257-L264
2. The comment for `WRAPPED_ETHER_ID` should be `hexadecimal(ascii("Ether"))` instead of `hexadecimal(ascii("shETH"))`.
```solidity
82:    /// @notice this is the oceanId used for shETH
83:    /// @dev hexadecimal(ascii("shETH"))
84:    uint256 public immutable WRAPPED_ETHER_ID;
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L82-L84