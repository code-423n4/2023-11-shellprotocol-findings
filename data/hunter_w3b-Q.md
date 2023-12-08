## [L-01] The Ocean::onERC721Received() is marked view but references internal state `_ERC721InteractionStatus`

The onERC721Received function is incorrectly marked as view, even though it references and relies on an internal state
variable `_ERC721InteractionStatus`.

A view function is supposed to only read contract state, not modify it. However, this function checks and behaves
differently based on the value of `_ERC721InteractionStatus`.

**Steps to Reproduce**

1. The function onERC721Received is defined with the view modifier
2. It contains an if statement that checks the value of `_ERC721InteractionStatus`
3. The return value and behavior depends on this state variable

The function reads and depends on the value of `_ERC721InteractionStatus`, violating the view semantics.

**Impact**

- The function can have unintended side effects by modifying state despite being marked view
- Calls to this function may not be treated as pure "reads" by wallets and fail
- State could become inconsistent if multiple callers read/modify state concurrently

```solidity
File: src/ocean/Ocean.sol

310        if (_ERC721InteractionStatus == INTERACTION) {

338        if (_ERC1155InteractionStatus == INTERACTION) {
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L338

**Recommendations**

1. Remove the view modifier from the function

## [L-02] The `Ocean::_unpackInteractionTypeAndAddress` function performs explicit type casts that could fail

The `_unpackInteractionTypeAndAddress` function performs explicit type casts that could fail or cause overflow issues
without validation.

No validation is performed, failing casts could lead to bugs.

**Impact**

- Incorrect type or address values used downstream
- Potentially corrupted application state
- Possible reverts or unintended behavior

**Recommendations**

1. Add validation checks for casts

```solidity
File: src/ocean/Ocean.sol

682    function _unpackInteractionTypeAndAddress(Interaction memory interaction)
683     internal
684     pure
685     returns (InteractionType interactionType, address externalContract)
686 {
687     bytes32 interactionTypeAndAddress = interaction.interactionTypeAndAddress;
688     interactionType = InteractionType(uint8(interactionTypeAndAddress[0]));
689     externalContract = address(uint160(uint256(interactionTypeAndAddress)));
690 }
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L682-L690 Please let me know if any

## [L-03] Using > 0 instead of != 0 may causes issues at low values due to rounding.

The amount comparison logic uses > 0 instead of != 0 which could cause problems at very low token amounts due to
rounding errors.

**Steps to Reproduce**

1. Execute an interaction that results in a very small output amount
2. The amount is checked with > 0 instead of != 0
3. Due to rounding, an amount like 0.00000001 could be rounded to 0

Amounts rounded to zero would not be detected as > 0.

**Impact**

- Very small balances could incorrectly be treated as zero
- Transactions with tiny amounts may fail or have unintended effects

**Recommendations**

1. Use != comparison for all amount checks

```solidity
File: src/ocean/Ocean.sol

419        if (inputAmount > 0) {

426        if (outputAmount > 0) {

527                if (inputAmount > 0) {

533                if (outputAmount > 0) {

1009        if (_isNotTokenOfPrimitive(inputToken, primitive) && (inputAmount > 0)) {

1043        if (_isNotTokenOfPrimitive(outputToken, primitive) && (outputAmount > 0)) {

1084        if (truncated > 0) {

1167        if (amount > 0) {
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L419
