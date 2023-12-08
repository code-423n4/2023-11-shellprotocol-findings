|  | issues | instance |
|---|---|---|
|[L-01]| use of time lock | 1 |
|[NC-02]|Separate Concerns| 1 |

DETAILS
[LC-01] Use of time lock
The `changeUnwrapFee` function does handle a time lock for managing fee changes, but could also include a time lock to prevent the fee from being changed too frequently. This could be done by storing the timestamp of the last fee change and checking that a certain amount of time has passed before allowing the fee to be changed again. This ensures that the fee can only be changed after a certain amount of time has passed since the last change also ensuring code follows best practice.
```
solidity
function changeUnwrapFee(uint256 nextUnwrapFeeDivisor) external override onlyOwner {
        /// @notice as the divisor gets smaller, the fee charged gets larger
        if (MIN_UNWRAP_FEE_DIVISOR > nextUnwrapFeeDivisor) revert();
```
[NC-02] Separate Concerns
The `changeUnwrapFee` function should separate the concern of changing the fee from the concern of emitting an event and updating the fee. This could be done by moving the emission of the event and the update of the fee to a separate function. As it enables a less cloggy codebase.
```
solidity
 function changeUnwrapFee(uint256 nextUnwrapFeeDivisor) external override onlyOwner {
        /// @notice as the divisor gets smaller, the fee charged gets larger
        if (MIN_UNWRAP_FEE_DIVISOR > nextUnwrapFeeDivisor) revert();
        emit ChangeUnwrapFee(unwrapFeeDivisor, nextUnwrapFeeDivisor, msg.sender);
        unwrapFeeDivisor = nextUnwrapFeeDivisor;
    }
```