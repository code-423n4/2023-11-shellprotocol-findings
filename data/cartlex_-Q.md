| ID | Description | Severity |
| :-: | - | :-: |
| [L-01](#l-01-owner-can-renounce-ownership) | Owner can renounce ownership. | Low |
| [NC-01](#nc-01-named-custom-errors-should-be-used) | Named custom errors should be used. | Non-critical |
| [NC-02](#nc-02-mistake-in-the-comments) | Mistake in the comments. | Non-critical |
| [NC-03](#nc-03-array-length-not-checked) | Array length not checked. | Non-critical |
| [NC-04](#nc-04-supportsinterface-should-be-edit) | `supportsInterface()` should be edit. | Non-critical |

# [L-01] Owner can renounce ownership.
## Description
`Ocean.sol` contract inherited from `OceanERC1155.sol` contract which in turn inherited from `Ownable`.
In `Ownable.sol` contract there is a `renounceOwnership()` which can be used by the owner to transfer the ownership to zero address, hence an `Ocean` contract can left without an owner and functions that used `onlyOwner` modifier can't be used.

```solidity
function renounceOwnership() public virtual onlyOwner {
        _transferOwnership(address(0));
}
```

## Recommended Mitigation Steps
Consider to override this function and make a revert to not allow the owner to renounce the ownership.

```diff
++  function renounceOwnership() public override onlyOwner {
++      revert OperationNotAllowed();
++  }
```
[https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L198](Ocean.sol#L198)


# [NC-01] Named custom errors should be used.
## Description
In `Ocean.sol` contract there is an unnamed custom erorr used.

```solidity
if (MIN_UNWRAP_FEE_DIVISOR > nextUnwrapFeeDivisor) revert(); // @audit [NC] unnamed error
```

## Recommended Mitigation Steps
Consider to use named custom errors to clarify user expirience.


# [NC-02] Mistake in the comments.

## Description
In the natspec comments to `forwardedDoInteraction()` function there is a mistake. It says that this function calls `_doMultipleInteractions()` function, however it calls `_doInteraction()` function.

```solidity
/**
     * @notice Execute interactions `interactions` on behalf of `userAddress`
     * @notice Does not need ids because a single interaction does not require
     *  the overhead of the intra-transaction accounting system
     * @dev MUST HAVE onlyApprovedForwarder modifer.
     * @dev call to _doMultipleInteractions() forwards the userAddress // @audit `_doInteraction()`, not `_doMultipleInteractions()`
     * @param interaction Executed to produce a set of balance updates
     * @param userAddress interactions are executed on behalf of this address
     */
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L252

## Recommended Mitigation Steps
Consider to change the comments.


# [NC-03] Array length not checked.

## Description
In `doMultipleInteractions()` input parameters are two arrays, however there is no check that their lenght is equal.

```solidity
function doMultipleInteractions(
    Interaction[] calldata interactions,
    uint256[] calldata ids
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L230-L231

## Recommended Mitigation Steps
Consider to check that length of input arrays is equal.


# [NC-04] `supportsInterface()` should be edit.

## Description
`Ocean.sol` contract inherit from `IERC721Receiver.sol`, however in `supportsInterface()` function there is no condition that interfaceId is equal to type of `IERC721Recevier`.

```solidity
function supportsInterface(bytes4 interfaceId) public view virtual override(OceanERC1155, IERC165) returns (bool) {
    return interfaceId == type(IERC1155Receiver).interfaceId || super.supportsInterface(interfaceId);
}
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L305-L307

## Recommended Mitigation Steps
Consider to implement the next changes:

```diff
function supportsInterface(bytes4 interfaceId) public view virtual override(OceanERC1155, IERC165) returns (bool) {
    return interfaceId == type(IERC1155Receiver).interfaceId
++  || interfaceId == type(IERC721Receiver).interfaceId
    || super.supportsInterface(interfaceId);
}
```