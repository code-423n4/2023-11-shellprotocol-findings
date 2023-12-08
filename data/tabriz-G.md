## [G‑01] Avoid contract existence checks by using low level calls
Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence

There are 37 instances of this issue:
```
 /// @audit increaseBalanceDelta() balanceDeltas.increaseBalanceDelta(WRAPPED_ETHER_ID, msg.value);

```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L478C11-L478C77

```
 /// @audit getBalanceDelta()
specifiedAmount = balanceDeltas.getBalanceDelta(interactionType, specifiedToken);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L516C21-L516C102

```
 /// @audit decreaseBalanceDelta() balanceDeltas.decreaseBalanceDelta(inputToken, inputAmount);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L529C19-L529C81

```
 /// @audit increaseBalanceDelta()
balanceDeltas.increaseBalanceDelta(outputToken, outputAmount);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L535C20-L535C83

```
/// @audit createMintAndBurnArrays()
 (mintIds, mintAmounts, burnIds, burnAmounts) = balanceDeltas.createMintAndBurnArrays();

```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L547C12-L548C1

```
 /// @audit computeOutputAmount()
 increaseBalanceDelta()IOceanPrimitive(primitive).computeOutputAmount(inputToken, outputToken, inputAmount, userAddress, metadata);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L760C10-L760C121

```
   /// @audit computeInputAmount()
IOceanPrimitive(primitive).computeInputAmount(inputToken, outputToken, outputAmount, userAddress, metadata);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L801C10-L801C121

```
 /// @audit safeTransferFrom()
SafeERC20.safeTransferFrom(IERC20(tokenAddress), userAddress, address(this), transferAmount);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L836C12-L836C106

```
/// @audit decimals()
  try IERC20Metadata(tokenAddress).decimals() returns (uint8 decimals) {
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L865C7-L865C79

```
 /// @audit safeTransfer() SafeERC20.safeTransfer(IERC20(tokenAddress), userAddress, transferAmount);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L875C11-L875C87

```
 /// @audit safeTransferFrom()
 increase()IERC721(tokenAddress).safeTransferFrom(userAddress, address(this), tokenId);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L891C8-L891C85

```
 /// @audit safeTransferFrom()
 IERC721(tokenAddress).safeTransferFrom(address(this), userAddress, tokenId);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L904C8-L904C85

```
 /// @audit safeTransferFrom() IERC1155(tokenAddress).safeTransferFrom(userAddress, address(this), tokenId, amount, "");
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L931C7-L931C98

```
 /// @audit safeTransferFrom() IERC1155(tokenAddress).safeTransferFrom(address(this), userAddress, tokenId, amountRemaining, "");
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L968C7-L968C107

```
 /// @audit transfer()
payable(userAddress).transfer(transferAmount);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L982C9-L982C55

```
 /// @audit decimals()
  decimals[yToken] = IERC20Metadata(yTokenAddress).decimals();
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L88C7-L88C69

```
/// @audit decimals()
   decimals[lpTokenId] = IERC20Metadata(primitive_).decimals();
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L93

```
 /// @audit doInteraction() IOceanInteractions(ocean).doInteraction(interaction);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L113C7-L113C62

```
 /// @audit doInteraction()
 IOceanInteractions(ocean).doInteraction(interaction);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L132C8-L132C62

```
 /// @audit exchange() ICurve2Pool(primitive).exchange(indexOfInputAmount, indexOfOutputAmount, rawInputAmount, 0);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L164C15-L164C109

```
/// @audit add_liquidity()
 rawOutputAmount = ICurve2Pool(primitive).add_liquidity(inputAmounts, 0);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L168C12-L168C85

```
/// @audit remove_liquidity_one_coin()
 rawOutputAmount = ICurve2Pool(primitive).remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L170C12-L170C120

```
 /// @audit approve() IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);
```
 https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L190
 ```
	 /// @audit approve() IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L191

```
 /// @audit doInteraction() IOceanInteractions(ocean).doInteraction{ value: amount }(interaction);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L129C11-L129C83

```
 /// @audit doInteraction()
 IOceanInteractions(ocean).doInteraction(interaction);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L138C12-L138C66

```
 /// @audit doInteraction()
 IOceanInteractions(ocean).doInteraction(interaction);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L168C8-L168C62

```
/// @audit exchange()
  ICurveTricrypto(primitive).exchange{ value: inputToken == zToken ? rawInputAmount : 0 }(
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L201C11-L201C101

```
 /// @audit add_liquidity()
 ICurveTricrypto(primitive).add_liquidity(inputAmounts, 0);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L210

```
 /// @audit remove_liquidity_one_coin() ICurveTricrypto(primitive).remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L214C15-L214C110

```
/// @audit withdraw()
 IWETH(underlying[zToken]).withdraw(
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L215

```
/// @audit balanceof()
IERC20Metadata(underlying[zToken]).balanceOf(address(this)) - wethBalance
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L216C21-L216C94

```
 /// @audit remove_liquidity_one_coin() ICurveTricrypto(primitive).remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L219C15-L219C110

```
   /// @audit approve() IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L242C5-L242C72

```
   /// @audit approve()
   IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L243C6-L243C76

```
/// @audit balanceOf()
return IERC20Metadata(tokenAddress).balanceOf(address(this));
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L253C13-L253C74

```
/// @audit unwrapFeeDivisor()
  uint256 unwrapFee = inputAmount / IOceanInteractions(ocean).unwrapFeeDivisor();
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L70C7-L70C88

## [G‑02] Use custom errors rather than revert()/require() strings to save gas
Custom errors are available from solidity version 0.8.4. Custom errors save ~50 gas each time they’re hit by avoiding having to allocate and store the revert string. Not defining the strings also save deployment gas

There are 8 instances of this issue:
```
  if (MIN_UNWRAP_FEE_DIVISOR > nextUnwrapFeeDivisor) revert();
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L198C7-L198C69


```
  revert NO_DECIMAL_METHOD();
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L840C11-L840C40

```
 revert NO_DECIMAL_METHOD();
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L878C12-L878C40
```
  if (tokenAddress == address(this)) revert NO_RECURSIVE_UNWRAPS();
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L964C7-L964C74

```
 revert INVALID_COMPUTE_TYPE();
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L217C12-L217C43

```
 revert INVALID_COMPUTE_TYPE();
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L287C12-L287C43

```
     require(msg.sender == ocean);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L39C4-L39C38

```
 revert();
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L93C8-L93C18

## [G-03] Empty blocks should be removed or emit 

There is 1 instance of this issue:
```
 fallback() external payable { }
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L291C4-L291C36

## [G-04] Caching storage values in memory
The code can be optimized by minimizing the number of SLOADs.

SLOADs are expensive (100 gas after the 1st one) compared to MLOADs/MSTOREs (3 gas each). Storage values read multiple times should instead be cached in memory the first time (costing 1 SLOAD) and then read from this cache to avoid multiple SLOADs.

There are 6 instances of this issue:
```
  if (tokenAddress == address(this)) revert NO_RECURSIVE_UNWRAPS();
        uint256 feeCharged = _calculateUnwrapFee(amount);
        uint256 amountRemaining = amount - feeCharged;
        _grantFeeToOcean(oceanId, feeCharged);
        IERC1155(tokenAddress).safeTransferFrom(address(this), userAddress, tokenId, amountRemaining, "");
        emit Erc1155Unwrap(tokenAddress, tokenId, amount, feeCharged, userAddress, oceanId);
    }
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L964C7-L970C6

```
  function _etherUnwrap(uint256 amount, address userAddress) private {
        uint256 feeCharged = _calculateUnwrapFee(amount);
        _grantFeeToOcean(WRAPPED_ETHER_ID, feeCharged);
        uint256 transferAmount = amount - feeCharged;
        payable(userAddress).transfer(transferAmount);
        emit EtherUnwrap(transferAmount, feeCharged, userAddress);
    }
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L978C3-L984C6

```
 (transferAmount, truncated) = _convertDecimals(NORMALIZED_DECIMALS, decimals, amount);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L1082C8-L1082C95

```
 transferAmount += 1;
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L1092C12-L1092C33

```
 if (outputToken == zToken) {
                uint256 wethBalance = IERC20Metadata(underlying[zToken]).balanceOf(address(this));
                ICurveTricrypto(primitive).remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);
                IWETH(underlying[zToken]).withdraw(
                    IERC20Metadata(underlying[zToken]).balanceOf(address(this)) - wethBalance
                );
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L212C12-L217C19

```
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
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L147C7-L159C6

## [G-05] ++numMinted costs less gas compared to numMinted += 1
This one isn’t in a for-loop and isn’t covered by c4udit

There is 1 instance of this issue:
```
 transferAmount += 1;
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L1092C12-L1092C33

## [G-06] OR in if-condition can be rewritten to two single if conditions
Refactoring the if-condition in a way it won’t be containing the || operator will save more gas.

This is a simple forge test, which demonstrates gas usage for both versions:
```
function testIfWithOR(
        int128 py_init,
        int128 px_init,
        int128 py_final,
        int128 px_final,
        uint256 duration
    ) public returns (uint) { 
        
        if (py_init >= MAX_PRICE_VALUE || py_final >= MAX_PRICE_VALUE) return 1;
        if (px_init <= MIN_PRICE_VALUE || px_final <= MIN_PRICE_VALUE) return 1;
    }
   function testIfWithoutOr(
        int128 py_init,
        int128 px_init,
        int128 py_final,
        int128 px_final,
        uint256 duration
    ) public returns (uint) { 
        
        if (py_init >= MAX_PRICE_VALUE) return 1;
        if (py_final >= MAX_PRICE_VALUE) return 1;
        if (px_init <= MIN_PRICE_VALUE) return 1;
        if (px_final <= MIN_PRICE_VALUE) return 1;
    }
```

```
Gas:testIfWithOR(int128,int128,int128,int128,uint256):(uint256) (runs: 256, μ: 914, ~: 926)
Gas:testIfWithoutOr(int128,int128,int128,int128,uint256):(uint256) (runs: 256, μ: 844, ~: 855)
```

There are 9 instances of this issue:
```
  if (interactionType == InteractionType.WrapErc20 || interactionType == InteractionType.UnwrapErc20) {
            specifiedToken = _calculateOceanId(externalContract, 0);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L709C7-L710C69

```
 } else if (
            interactionType == InteractionType.WrapErc721 || interactionType == InteractionType.WrapErc1155
                || interactionType == InteractionType.UnwrapErc721 || interactionType == InteractionType.UnwrapErc1155
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L711C8-L713C119

```
if (((inputToken == xToken) && (outputToken == yToken)) || ((inputToken == yToken) && (outputToken == xToken)))
        {
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L209C9-L210C10

```
  } else if (((inputToken == xToken) || (inputToken == yToken)) && (outputToken == lpTokenId)) {
            return ComputeType.Deposit;
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L212C7-L213C40

```
  } else if ((inputToken == lpTokenId) && ((outputToken == xToken) || (outputToken == yToken))) {
            return ComputeType.Withdraw;
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L214C7-L215C41

```
 if (action == ComputeType.Swap) {
            bool useEth = inputToken == zToken || outputToken == zToken;
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L198C8-L199C73

```
if (
            ((inputToken == xToken && outputToken == yToken) || (inputToken == yToken && outputToken == xToken))
                || ((inputToken == xToken && outputToken == zToken) || (inputToken == zToken && outputToken == xToken))
                || ((inputToken == yToken && outputToken == zToken) || (inputToken == zToken && outputToken == yToken))
        ) {
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L272C9-L276C12

```
   } else if (
            ((inputToken == xToken) || (inputToken == yToken) || (inputToken == zToken)) && (outputToken == lpTokenId)
        ) {
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L278C6-L280C12

```
  } else if (
            (inputToken == lpTokenId) && ((outputToken == xToken) || (outputToken == yToken) || (outputToken == zToken))
        ) {
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L282C7-L284C12

## [G‑07]	Consider using ERC721A instead of ERC721

There are 3 instances of this issue:
```
import { IERC721 } from "@openzeppelin/contracts/token/ERC721/IERC721.sol";
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L12C1-L12C76

```
  IERC721(tokenAddress).safeTransferFrom(userAddress, address(this), tokenId);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L891C7-L891C85

```
    IERC721(tokenAddress).safeTransferFrom(address(this), userAddress, tokenId);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L904C5-L904C85