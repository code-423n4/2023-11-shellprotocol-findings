

## Gas Optimization 


| No | Instance | Description |

| [G-01] | The accounting engine of the shell protocol|
| [G-02] |  Adapter that enables integration with the curve 2 pool|
| [G-03] | Adapter that enables integration with the curve tricrypto pool.|
| [G-04] | Helper contract for the adapters|



### Code Descriptoin in Solidity


### [G-01] The accounting engine of the shell protocol

```solidity
file: oceans.sol
```
   _decreaseBalanceOfPrimitive(primitive, outputToken, outputAmount);

        inputAmount =
            IOceanPrimitive(primitive).computeInputAmount(inputToken, outputToken, outputAmount, userAddress, metadata);

        _increaseBalanceOfPrimitive(primitive, inputToken, inputAmount);

        emit ComputeInputAmount(primitive, inputToken, outputToken, inputAmount, outputAmount, userAddress);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L798-L805

```



### [G-02] Adapter that enables integration with the curve 2 pool.

```solidity
file: Curve2PoolAdapter.sol
```
 rawOutputAmount =
                ICurve2Pool(primitive).exchange(indexOfInputAmount, indexOfOutputAmount, rawInputAmount, 0);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L163-L164
```



####  [G-03] Adapter that enables integration with the curve tricrypto pool.

```solidity
file: CurveTricryptoAdapter.sol
```
contract CurveTricryptoAdapter is OceanAdapter {
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L25
```




#### [G-4] Helper contract for the adapters

```solidity
file: OceanAdapter.sol
```
abstract contract OceanAdapter extends IOceanPrimitive {
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L14
```
