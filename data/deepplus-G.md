## Use != 0 instead of > 0 for unsigned integer comparison

When dealing with unsigned integer types, comparisons with != 0 are cheaper then with > 0.

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L419
https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L426
https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L527
https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L533
https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L1084
https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L1167

## Replace the first condition of `_determineComputeType` function with `inputToken != lpToken && inputToken != lpToken && inputToken != outputToken`

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L273-L275
```solidity
        if (
            ((inputToken == xToken && outputToken == yToken) || (inputToken == yToken && outputToken == xToken))
                || ((inputToken == xToken && outputToken == zToken) || (inputToken == zToken && outputToken == xToken))
                || ((inputToken == yToken && outputToken == zToken) || (inputToken == zToken && outputToken == yToken))
        ) {
            return ComputeType.Swap;
        } else if (
            ((inputToken == xToken) || (inputToken == yToken) || (inputToken == zToken)) && (outputToken == lpTokenId)
        ) {
            return ComputeType.Deposit;
        } else if (
            (inputToken == lpTokenId) && ((outputToken == xToken) || (outputToken == yToken) || (outputToken == zToken))
        ) {
            return ComputeType.Withdraw;
        } else {
            revert INVALID_COMPUTE_TYPE();
        }
```