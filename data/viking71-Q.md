### Use `require` instead of `assert` for error handling

**File(s)**: [`Ocean.sol`](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol)

**Description**: `assert()` should be avoided past solidity version 0.8.0 as its [documentation](https://docs.soliditylang.org/en/v0.8.23/control-structures.html#panic-via-assert-and-error-via-require) states that "The assert function creates an error of type Panic(uint256). ... Properly functioning code should never create a Panic, not even on invalid external input. If this happens, then there is a bug in your contract which you should fix".
`_determineTransferAmount` (https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L1100), `_executeInteraction` (https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L667) and `_getSpecifiedToken` (https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L721) functions in the contract use assert function. 

**Recommendation(s)**: Consider using require for error handling instead of assert.