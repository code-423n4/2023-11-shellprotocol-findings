### Low-01: Ocean.sol `_doMultipleInteractions()` doesn't check for duplicated `ids` input array (Note: not submitted in bot report)

In Ocean.sol `_doMultipleInteractions()` both the interactions array(`Interaction[] calldata`) and ocean Ids(`uint256[] calldata`) array required for the interactions are passed by callers. Note that interactions array and ids array can be of different lengths and the interactions array can contain duplicated interaction (e.g. adding the same liquidity twice, etc), however, Ids array shouldn't contain duplicated elements. 

When Ids array contain the same OceanId twice, only the first index of the OceanId in the array will be correctly modified, the other indexes of duplicated OceanId will be left at default 0 value. The impact is every time all the indexes will be looped through in a for-loop `_findIndexOfTokenId()`, `_copyDeltasToMintAndBurnArrays()` from BalanceDelta.sol, this will increase unnecessary number of iterations, every time a duplicated id is passed to `_doMultipleInteractions()`. The more ocean ids involved in the transaction, the more wasteful iterations of the duplicated ids it will generate.

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

### Low-02: In Curve2PoolAdapter.sol `constructor()`, there is unnecessary external call to approve `primitive_` to spend `primitive_`
In src/adapters/Curve2PoolAdapter.sol `_approveToken()`, for a given `tokenAddress` input, this will approve both ocean and primitive to spend `token` on behalf of the adaptor. 

In the `constructor`, `_approveToken()` is called for `xTokenAddress`, `yTokenAddress`, and also `primitive_`. In the last case, `_approveToken()` will approve `primitive_` to spend `primitive_` on behalf of the adapter. This is an unnecessary external call, because `primitive_` will be curve usdc-usdt pool address, and curve2Pool will not need to call itself with `transferFrom` function selector, since when removing liquidity, Curve usdc-usdt pool will directly reduce the Curve2PoolAdapter's balance.

```solidity
//src/adapters/Curve2PoolAdapter.sol
    constructor(address ocean_, address primitive_) OceanAdapter(ocean_, primitive_) {
...
        lpTokenId = _calculateOceanId(primitive_, 0);
        underlying[lpTokenId] = primitive_;
        decimals[lpTokenId] = IERC20Metadata(primitive_).decimals();
        //@audit _approveToken will also approve `primitive_` to spend `primitive_` 
|>      _approveToken(primitive_);
```
(https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L94)

```solidity
//src/adapters/Curve2PoolAdapter.sol
    function _approveToken(address tokenAddress) private {
        IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);
        //@audit when _approveToken(primitive_) is called in constructor, this make an unnecessary external call to curve usdc-usdt pool.
|>      IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);
    }
```

```vyper
//Curve usdc-usdt pool compatible with Curve2PoolAdapter.sol
def remove_liquidity_one_coin(
    _burn_amount: uint256,
    i: int128,
    _min_received: uint256,
    _receiver: address = msg.sender,
) -> uint256:
...
|>  self.balanceOf[msg.sender] -= _burn_amount
```
(https://arbiscan.io/token/0x7f90122bf0700f9e7e1f688fe926940e8839f353#code)
As seen above, when removing liquidity, Curve2PoolAdapter's `primitive_` balance will be directly subtracted without the need for transferFrom `primitive_`. So no approving `primitive_` for `primitve_` is needed.

**Recommendations:**
Because in this case `primitive_` is the same as Lp token address, consider in Curve2PoolAdapter.sol `_approveToken()` add a bypass such that only when `tokenAddress !=primitive`,`IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max) will be called. 


### Low-03: `_convertDecimals()` are declared in both OceanAdapter.sol and Ocean.sol with minor differences, consider refactoring this function as part of a library contract (Note: Not included in bot report)
In Ocean.sol and OceanAdapter.sol `_convertDecimals()` are declared twice with almost identical implementations except that in Ocean.sol, `truncatedAmount` is assigned and returned, while in OceanAdapter.sol `truncatedAmount` is not handled and returned.

```solidity
//src/adapters/OceanAdapter.sol
    function _convertDecimals(
        uint8 decimalsFrom,
        uint8 decimalsTo,
        uint256 amountToConvert
    )
        internal
        pure
        returns (uint256 convertedAmount)
    {
        if (decimalsFrom == decimalsTo) {
            // no shift
            convertedAmount = amountToConvert;
        } else if (decimalsFrom < decimalsTo) {
            // Decimal shift left (add precision)
            uint256 shift = 10 ** (uint256(decimalsTo - decimalsFrom));
            convertedAmount = amountToConvert * shift;
        } else {
            // Decimal shift right (remove precision) -> truncation
            uint256 shift = 10 ** (uint256(decimalsFrom - decimalsTo));
            convertedAmount = amountToConvert / shift;
        }
    }
```
(https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L138-L159)
```solidity
//src/ocean/Ocean.sol
    function _convertDecimals(
        uint8 decimalsFrom,
        uint8 decimalsTo,
        uint256 amountToConvert
    )
        internal
        pure
        returns (uint256 convertedAmount, uint256 truncatedAmount)
    {
        if (decimalsFrom == decimalsTo) {
            // no shift
            convertedAmount = amountToConvert;
            truncatedAmount = 0;
        } else if (decimalsFrom < decimalsTo) {
            // Decimal shift left (add precision)
            uint256 shift = 10 ** (uint256(decimalsTo - decimalsFrom));
            convertedAmount = amountToConvert * shift;
            truncatedAmount = 0;
        } else {
            // Decimal shift right (remove precision) -> truncation
            uint256 shift = 10 ** (uint256(decimalsFrom - decimalsTo));
            convertedAmount = amountToConvert / shift;
            truncatedAmount = amountToConvert % shift;
        }
    }
```
(https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L1123-L1145)

**Recommendation:**
Consider moving decimals conversion into a library contract. In OceanAdapter.sol, the same `_convertDecimals()` from Ocean.sol can still be used. 

### NC-01 Some functions in abstract OceanAdapter can be refactored
OceanAdapter.sol is an abstract contract intended to be overwritten by child contract with specific implementations suited for corresponding external liquidity pools. For example, Curve2PoolAdapter.sol overwrites OceanAdapter.sol to wrap curve usdc-usdt pool. In this case, OceanAdapter should avoid specifying implementation that is not general and refactor these into the child contract.

One example is `computeInputAmount()` in OceanAdapter.sol. Current `computeInputAmount()` has `revert()` that could be implemented in Child contracts instead.

```solidity
//src/adapters/OceanAdapter.sol
    function computeInputAmount(
        uint256 inputToken,
        uint256 outputToken,
        uint256 outputAmount,
        address userAddress,
        bytes32 maximumInputAmount
    )
       external
        override
        onlyOcean
        returns (uint256 inputAmount)
    {
        revert();
    }
```
(https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L93)

**Recommendations:**
Consider moving `revert()` to be implemented in the child contract.

### NC-02 Curve2PoolAdapter.sol doesn't work with all Curve2Pools currently implemented by Curve
Based on the code comment, the current implementation of Curve2PoolAdapter.sol is designed for curve usdc-usdt pool. However, Curve2PoolAdapter.sol will not work with current Curve2Pools that use a LpToken contract separate from the pool contract.

For example, current implementation will not work with Curve Crv-Eth pool. See [here](https://etherscan.io/address/0x8301ae4fc9c624d1d396cbdaa1ed877821d7c511#code). Curve Crv-Eth pool uses a separate LpToken contract -[Curve Crv-Eth token](https://etherscan.io/address/0xEd4064f376cB8d68F770FB1Ff088a3d0F3FF5c4d).

```vyper
//Curve Crv-Eth pool
# These addresses are replaced by the deployer
//@audit in Crv-Eth pool, a separate hardcoded LpToken address is defined, different from pool address itself.
token: constant(address) = 0xEd4064f376cB8d68F770FB1Ff088a3d0F3FF5c4d
coins: constant(address[N_COINS]) = [
    0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2,
    0xD533a949740bb3306d119CC777fa900bA034cd52]
```
(https://etherscan.io/address/0x8301ae4fc9c624d1d396cbdaa1ed877821d7c511#code)

```viper
//Curve Crv-Eth LpToken
"""
@title Curve LP Token
@author Curve.Fi
@notice Base implementation for an LP token provided for
        supplying liquidity to `StableSwap`
@dev Follows the ERC-20 token standard as defined at
     https://eips.ethereum.org/EIPS/eip-20
"""
```
(https://etherscan.io/address/0xEd4064f376cB8d68F770FB1Ff088a3d0F3FF5c4d#code)

In this case, Curve2PoolAdapter.sol will not work because it only defines xToken, yToken and primitive as input or output token, when it should be xToken, yToken and LpToken as input or output token.

```solidity
//src/adapters/Curve2PoolAdapter.sol
    constructor(address ocean_, address primitive_) OceanAdapter(ocean_, primitive_) {
...
      address xTokenAddress = ICurve2Pool(primitive).coins(0);
        xToken = _calculateOceanId(xTokenAddress, 0);
        underlying[xToken] = xTokenAddress;
        decimals[xToken] = IERC20Metadata(xTokenAddress).decimals();
        _approveToken(xTokenAddress);

...
        address yTokenAddress = ICurve2Pool(primitive).coins(1);
        yToken = _calculateOceanId(yTokenAddress, 0);
        indexOf[yToken] = int128(1);
        underlying[yToken] = yTokenAddress;
        decimals[yToken] = IERC20Metadata(yTokenAddress).decimals();
        _approveToken(yTokenAddress);
...
        //@audit this is incorrect when curve2pool use a separate lpToken contract different from primitive contract such as crv-eth pool.
|>      lpTokenId = _calculateOceanId(primitive_, 0);
        underlying[lpTokenId] = primitive_;
        //@audit this will also revert due to primitive_ has no decimal methods
|>      decimals[lpTokenId] = IERC20Metadata(primitive_).decimals();
        _approveToken(primitive_);
```
(https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L91-L94)

In addition, development will fail due to primitive contract has no decimal method since it's not intended to be ERC20.

**Recommendations:**
(1) Either develop a different version of Curve2PoolAdapter that work with Curve pool that has a separate LpToken;
(2) Or revise current Curve2PoolAdapter to always use xToken, yToken and LpToken. And in the constructor, bypass when calling `.token()` method on curve pool fails which indicate it doesn't use a separate LPToken.

