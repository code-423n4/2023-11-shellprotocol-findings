# GAS OPTIMIZATION

# SUMMARY
|      |  issue  |  instance  |
|------|---------|------------|
|[G‑01]|Refactor external/internal function to avoid unnecessary External Calls|17|
|[G‑02]|Can Make The Variable Outside The Loop To Save Gas |2|
|[G‑03]|Sort Solidity operations using short-circuit mode|2|
|[G‑04]|Use hardcode address instead address(this)|11|
|[G‑05]|Use constants instead of type(uintx).max|6|
|[G‑06]|Use assembly to perform efficient back-to-back calls|2|
|[G‑07]|When possible, use assembly instead of unchecked{++i}|1|
|[G‑08]|Avoid emitting storage values|1|
|[G‑09]|unchecked {} can be used on the division of two uints in order to save gas|1|
|[G‑10]|Optimize External Calls with Assembly for Memory Efficiency|14|
|[G‑11]|Using constants instead of enum can save gas|2|


## [G-01] Refactor external/internal function to avoid unnecessary External Calls
The functions below perform external calls that are previously performed in the functions that invoke them. We can refactor the external/internal functions so we could pass the cached external calls into them and avoid the extra external calls that would otherwise take place in the internal functions.

```solidity
File: adapters/Curve2PoolAdapter.sol
164   ICurve2Pool(primitive).exchange(indexOfInputAmount, indexOfOutputAmount, rawInputAmount, 0);

168   rawOutputAmount = ICurve2Pool(primitive).add_liquidity(inputAmounts, 0);

170   rawOutputAmount = ICurve2Pool(primitive).remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L164

To optimize the gas usage in the given function, you can cache the ICurve2Pool(primitive) contract instance at the beginning of the function. Here's an updated version of the code with the caching implemented:
```diff
    function primitiveOutputAmount(
        uint256 inputToken,
        uint256 outputToken,
        uint256 inputAmount,
        bytes32 minimumOutputAmount
    )
        internal
        override
        returns (uint256 outputAmount)
    {
+       ICurve2Pool curvePool = ICurve2Pool(primitive);  // Cache ICurve2Pool contract instance 

        uint256 rawInputAmount = _convertDecimals(NORMALIZED_DECIMALS, decimals[inputToken], inputAmount);

        ComputeType action = _determineComputeType(inputToken, outputToken);

        uint256 rawOutputAmount;

        // avoid multiple SLOADS
        int128 indexOfInputAmount = indexOf[inputToken];
        int128 indexOfOutputAmount = indexOf[outputToken];

        if (action == ComputeType.Swap) {
            rawOutputAmount =
-               ICurve2Pool(primitive).exchange(indexOfInputAmount, indexOfOutputAmount, rawInputAmount, 0);
+               curvePool.exchange(indexOfInputAmount, indexOfOutputAmount, rawInputAmount, 0);
        } else if (action == ComputeType.Deposit) {
            uint256[2] memory inputAmounts;
            inputAmounts[uint256(int256(indexOfInputAmount))] = rawInputAmount;
-           rawOutputAmount = ICurve2Pool(primitive).add_liquidity(inputAmounts, 0);
+           rawOutputAmount = curvePool.add_liquidity(inputAmounts, 0);
        } else {
-           rawOutputAmount = ICurve2Pool(primitive).remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);
+           rawOutputAmount = curvePool.remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);
        }

        outputAmount = _convertDecimals(decimals[outputToken], NORMALIZED_DECIMALS, rawOutputAmount);

        if (uint256(minimumOutputAmount) > outputAmount) revert SLIPPAGE_LIMIT_EXCEEDED();

        if (action == ComputeType.Swap) {
            emit Swap(inputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);
        } else if (action == ComputeType.Deposit) {
            emit Deposit(inputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);
        } else {
            emit Withdraw(outputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);
        }
    }

```


in _approveToken function IERC20Metadata(tokenAddress) is external call
```solidity
File: adapters/Curve2PoolAdapter.sol
        IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);
        IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L190-L191




in wrapToken function IOceanInteractions(ocean) is external call
```solidity
File: adapters/CurveTricryptoAdapter.sol
129  IOceanInteractions(ocean).doInteraction{ value: amount }(interaction);

138  IOceanInteractions(ocean).doInteraction(interaction);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L129



in primitiveOutputAmount function ICurveTricrypto(primitive) is external call
```solidity
File: adapters/CurveTricryptoAdapter.sol
201  ICurveTricrypto(primitive).exchange{ value: inputToken == zToken ? rawInputAmount : 0 }(

210  ICurveTricrypto(primitive).add_liquidity(inputAmounts, 0);

214  ICurveTricrypto(primitive).remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);

219  ICurveTricrypto(primitive).remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L201




in primitiveOutputAmount function IWETH(underlying[zToken]) is external call
```solidity
File: adapters/CurveTricryptoAdapter.sol
208   if (inputToken == zToken) IWETH(underlying[zToken]).deposit{ value: rawInputAmount }();

215   IWETH(underlying[zToken]).withdraw(
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L208




in primitiveOutputAmount function IERC20Metadata(underlying[zToken]) is external call
```solidity
File: adapters/CurveTricryptoAdapter.sol
213  uint256 wethBalance = IERC20Metadata(underlying[zToken]).balanceOf(address(this));

219  ICurveTricrypto(primitive).remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L213



in _approveToken function IERC20Metadata(tokenAddress) is external call

```solidity
File: adapters/CurveTricryptoAdapter.sol
        IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);
        IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L242-L243

## [G-02] Can Make The Variable Outside The Loop To Save Gas 
When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here's an example:

```
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total = 0;
        
        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }
        
        return total;
    }
}
```

There are 2 instances of this issue:


```solidity
File: ocean/Ocean.sol
508   uint256 specifiedToken = _getSpecifiedToken(interactionType, externalContract, interaction);

514   uint256 specifiedAmount;
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L508


## [G-03] Sort Solidity operations using short-circuit mode
Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 
//Sort operations with different gas costs as follows 
f(x) || g(y) 
f(x) && g(y)
```
There are 2 instances of this issue:

 _isNotTokenOfPrimitive(inputToken, primitive) function call is expensive then (inputAmount > 0)
```solidity
File: ocean/Ocean.sol
1009    if (_isNotTokenOfPrimitive(inputToken, primitive) && (inputAmount > 0)) {

1043    if (_isNotTokenOfPrimitive(outputToken, primitive) && (outputAmount > 0)) {
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L1009

## [G-04] Use hardcode address instead address(this)
it can be more gas-efficient to use a hardcoded address instead of the address(this) expression, especially if you need to use the same address multiple times in your contract.

The reason for this is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract's address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

Here's an example of how you can use a hardcoded address instead of address(this):

```
contract MyContract {
    address public immutable myAddress = 0x1234567890123456789012345678901234567890;
    #L
    function doSomething() public {
        // Use myAddress instead of address(this)
        require(msg.sender == myAddress, "Caller is not authorized");
        
        // Do something
    }
}
```
In the above example, we have a contract MyContract with a public address variable myAddress. Instead of using address(this) to retrieve the contract's address, we have pre-calculated and hardcoded the address in the variable. This can help to reduce the gas cost of our contract and make our code more efficient.

[References](https://book.getfoundry.sh/reference/forge-std/compute-create-address)

There are 11 instances of this issue:



```solidity
File: ocean/Ocean.sol
836:             SafeERC20.safeTransferFrom(IERC20(tokenAddress), userAddress, address(this), transferAmount);

891         IERC721(tokenAddress).safeTransferFrom(userAddress, address(this), tokenId);

904         IERC721(tokenAddress).safeTransferFrom(address(this), userAddress, tokenId);

929         if (tokenAddress == address(this)) revert NO_RECURSIVE_WRAPS();

931         IERC1155(tokenAddress).safeTransferFrom(userAddress, address(this), tokenId, amount, "");

964         if (tokenAddress == address(this)) revert NO_RECURSIVE_UNWRAPS();

968         IERC1155(tokenAddress).safeTransferFrom(address(this), userAddress, tokenId, amountRemaining, "");
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L836


```solidity
File: adapters/CurveTricryptoAdapter.sol
213          uint256 wethBalance = IERC20Metadata(underlying[zToken]).balanceOf(address(this));

216          IERC20Metadata(underlying[zToken]).balanceOf(address(this)) - wethBalance

251          return address(this).balance;

253          return IERC20Metadata(tokenAddress).balanceOf(address(this));
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L213

## [G-05] Use constants instead of type(uintx).max
Using constants instead of type(uintx).max can save gas in some cases because constants are precomputed at compile-time and can be reused throughout the code, whereas type(uintx).max requires computation at runtime

There are 6 instances of this issue:

```solidity
File: adapters/Curve2PoolAdapter.sol
        IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);
        IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L190-L191


```solidity
File: adapters/CurveTricryptoAdapter.sol
        IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);
        IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L242-L243


```solidity
File: ocean/Ocean.sol
102   uint256 constant GET_BALANCE_DELTA = type(uint256).max;

170    unwrapFeeDivisor = type(uint256).max;
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L102


## [G-06] Use assembly to perform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.
Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary.
[Reffrence](https://code4rena.com/reports/2023-05-ajna#g-10-use-assembly-to-perform-efficient-back-to-back-calls)

```solidity
File: adapters/Curve2PoolAdapter.sol
        IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);
        IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L190-L191

```solidity
File: adapters/CurveTricryptoAdapter.sol
        IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);
        IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L242-L243



## [G-07] When possible, use assembly instead of unchecked{++i}
You can also use unchecked{++i;} for even more gas savings but this will not check to see if i overflows. For best gas savings, use inline assembly, however, this limits the functionality you can achieve.

```
//loop with unchecked{++i}
function uncheckedPlusPlusI() public pure {
    uint256 j = 0;
    for (uint256 i; i < 10; ) {
        j++;
        unchecked {
            ++i;
        }
    }
}
```
Gas: 1329
```
//loop with inline assembly
function inlineAssemblyLoop() public pure {
    assembly {
        let j := 0
        for {
            let i := 0
        } lt(i, 10) {
            i := add(i, 0x01)
        } {
            j := add(j, 0x01)
        }
    }
}
```
Gas: 709


```solidity
File: ocean/Ocean.sol
        for (uint256 i = 0; i < _idLength;) {
            balanceDeltas[i] = BalanceDelta(ids[i], 0);
            unchecked {
                ++i;
            }
        }

```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L463-L469



## [G-08] Avoid emitting storage values
Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. We can avoid unecessary SLOADs by caching storage values that were previously accessed and emitting those cached values.

There are 1 instances of this issue:

```solidity
File: ocean/Ocean.sol
199    emit ChangeUnwrapFee(unwrapFeeDivisor, nextUnwrapFeeDivisor, msg.sender);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L199

## [G-09] unchecked {} can be used on the division of two uints in order to save gas

There are 1 instances of this issue:
```solidity
File: adapters/OceanAdapter.sol
157  convertedAmount = amountToConvert / shift;
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L157

## [G-10] Optimize External Calls with Assembly for Memory Efficiency
Using interfaces to make external contract calls in Solidity is convenient but can be inefficient in terms of memory utilization. Each such call involves creating a new memory location to store the data being passed, thus incurring memory expansion costs.

Inline assembly allows for optimized memory usage by re-using already allocated memory spaces or using the scratch space for smaller datasets. This can result in notable gas savings, especially for contracts that make frequent external calls.

Additionally, using inline assembly enables important safety checks like verifying if the target address has code deployed to it using extcodesize(addr) before making the call, mitigating risks associated with contract interactions.

There are 14 instances of this issue:
```solidity
File: adapters/Curve2PoolAdapter.sol
113   IOceanInteractions(ocean).doInteraction(interaction);

132   IOceanInteractions(ocean).doInteraction(interaction);

164   ICurve2Pool(primitive).exchange(indexOfInputAmount, indexOfOutputAmount, rawInputAmount, 0);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L164

```solidity
File: adapters/Curve2PoolAdapter.sol
        IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);
        IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L190-L191

```solidity
File: adapters/CurveTricryptoAdapter.sol
129  IOceanInteractions(ocean).doInteraction{ value: amount }(interaction);

138  IOceanInteractions(ocean).doInteraction(interaction);

168  IOceanInteractions(ocean).doInteraction(interaction);

242  IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);
243  IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L129

```solidity
File: ocean/Ocean.sol
891  IERC721(tokenAddress).safeTransferFrom(userAddress, address(this), tokenId);

904  IERC721(tokenAddress).safeTransferFrom(address(this), userAddress, tokenId);

931  IERC1155(tokenAddress).safeTransferFrom(userAddress, address(this), tokenId, amount, "");

968  IERC1155(tokenAddress).safeTransferFrom(address(this), userAddress, tokenId, amountRemaining, "");
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L891


## [G‑11] Using constants instead of enum can save gas
Enum is expensive and it is more efficient to use constants instead. An illustrative example of this approach can be found in the ReentrancyGuard.sol.


```solidity
File: adapters/Curve2PoolAdapter.sol
enum ComputeType {
    Deposit,
    Swap,
    Withdraw
}
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L10-L14


```solidity
File: adapters/CurveTricryptoAdapter.sol
enum ComputeType {
    Deposit,
    Swap,
    Withdraw
}
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L15-L19