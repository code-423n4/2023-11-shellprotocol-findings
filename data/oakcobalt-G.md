### Gas-01 `_doMultipleInteractions()` might cause wasteful for-loop iterations and can be optimized
In Ocean.sol `_doMultipleInteractions()`, the input `uint256[] calldata ids` array is not checked for duplication. When there are duplicated ids in the array, wasteful for-loop iterations will occur.

When Ids array contain the same OceanId twice, only the first index of the OceanId in the array will be correctly modified, the other indexes of duplicated OceanId will be left at default 0 value. The impact is every time all the indexes will be looped through in a for-loop `_findIndexOfTokenId()` and `_copyDeltasToMintAndBurnArrays` from BalanceDelta.sol, this will increase unnecessary number of iterations, every time a duplicated id is passed to `_doMultipleInteractions()`. The more ocean ids involved in the transaction, the more wasteful iterations of the duplicated ids it will generate.

```solidity
//src/ocean/Ocean.sol
    function _doMultipleInteractions(
        Interaction[] calldata interactions,
        uint256[] calldata ids,
        address userAddress
    )
...
        // Use the passed ids to create an array of balance deltas, used in
        // the intra-transaction accounting system.
        //@audit duplicated ids will create addition empty elements in balancesDeltas
|>      BalanceDelta[] memory balanceDeltas = new BalanceDelta[](ids.length);
        uint256 _idLength = ids.length;
        for (uint256 i = 0; i < _idLength;) {
            balanceDeltas[i] = BalanceDelta(ids[i], 0);
            unchecked {
                ++i;
            }
        }
...
        balanceDeltas.increaseBalanceDelta(WRAPPED_ETHER_ID, msg.value);
```
(https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L460)
```solidity
//src/ocean/BalanceDelta.sol
    function increaseBalanceDelta(
        BalanceDelta[] memory self,
        uint256 tokenId,
        uint256 amount
    )
...
|>      uint256 index = _findIndexOfTokenId(self, tokenId);
        self[index].delta += int256(amount);
        return;
```
(https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/BalanceDelta.sol#L85)
```solidity
//src/ocean/BalanceDelta.sol
    function _findIndexOfTokenId(BalanceDelta[] memory self, uint256 tokenId) private pure returns (uint256 index) {
|>      for (index = 0; index < self.length; ++index) {
            if (self[index].tokenId == tokenId) {
                return index;
            }
        }
```
(https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/BalanceDelta.sol#L308-L311)
```solidity
//src/ocean/BalanceDelta.sol
    function _copyDeltasToMintAndBurnArrays(
        BalanceDelta[] memory self,
        uint256[] memory mintIds,
        uint256[] memory mintAmounts,
        uint256[] memory burnIds,
        uint256[] memory burnAmounts
    )
...
         //@audit This will iterate through all BalanceDelta elements even those with duplicated Ids
|>       for (uint256 i = 0; i < self.length; ++i) {
            int256 delta = self[i].delta;
```
(https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/BalanceDelta.sol#L259)

**Recommendations:**
Consider check for duplicates in the initial ids array in `_doMultipleInteractions` and remove duplicated id element before declaring `BalanceDelta[]`. This removes unnecessary for-loop iterations later on in the transaction.