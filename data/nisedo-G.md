# Gas Optimization Report
All optimizations presented below were benchmarked using the protocol’s default test parameters via `forge test --gas-report`.

Total gas savings achieved by applying all the presented optimizations:
| Metric            | Savings     |
|-------------------|-------------|
| AVG Deployment Costs  | **-93,999**     |
| AVG Deployment Size   | **-58,883**     |


## G1 - Refactor Ocean::_determineTransferAmount

This optimization involves replacing `assert` statements with `if` statements in `Ocean::_determineTransferAmount`. This change improves gas efficiency by avoiding the use of `assert`, which is more costly in terms of gas usage.

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L1100-L1102

```solidity
File: src/ocean/Ocean.sol

            assert(normalizedTruncatedAmount == 0);
            assert(normalizedTransferAmount > amount);
            dust = normalizedTransferAmount - amount;
```

```diff
File: src/ocean/Ocean.sol

-           assert(normalizedTruncatedAmount == 0);
-           assert(normalizedTransferAmount > amount);
-           dust = normalizedTransferAmount - amount;

+           if (normalizedTruncatedAmount == 0){
+               if (normalizedTransferAmount > amount) {
+                   dust = normalizedTransferAmount - amount;
+               }
+           }
```

| Description                         | Before    | After     | Savings      |
|-------------------------------------|-----------|-----------|-----------------|
| Ocean.sol AVG Deployment Costs (Gas)| 4,103,602 | 4,100,802 | **-2,800 (-0.07%)** |
| Ocean.sol AVG Deployment Size (Gas) | 21,141    | 21,127    | **-14 (-0.07%)**    |

## G2 - Public immutable variables can be set to internal to save gas

Changing `public immutable` variables to `internal` results in significant gas savings during deployment.

### Curve2PoolAdapter.sol

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L55-L62

```solidity
File: src/adapters/Curve2PoolAdapter.sol

    /// @notice x token Ocean ID.
    uint256 public immutable xToken;


    /// @notice y token Ocean ID.
    uint256 public immutable yToken;


    /// @notice lp token Ocean ID.
    uint256 public immutable lpTokenId;
```

```diff
File: src/adapters/Curve2PoolAdapter.sol

    /// @notice x token Ocean ID.
-   uint256 public immutable xToken;
+   uint256 internal immutable xToken;


    /// @notice y token Ocean ID.
-   uint256 public immutable yToken;
+   uint256 internal immutable yToken;


    /// @notice lp token Ocean ID.
-   uint256 public immutable lpTokenId;
+   uint256 internal immutable lpTokenId;
```

| Description                                    | Before      | After       | Savings          |
|------------------------------------------------|-------------|-------------|---------------------|
| Curve2PoolAdapter.sol AVG Deployment Costs     | 1,291,528   | 1,255,043   | **-36,485 (-2.83%)**    |
| Curve2PoolAdapter.sol AVG Deployment Size      | 6,497       | 6,294       | **-203 (-3.13%)**       |

### CurveTricryptoAdapter.sol

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L60-L64

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

    /// @notice x token Ocean ID
    uint256 public immutable xToken;


    /// @notice y token Ocean ID
    uint256 public immutable yToken;
```

```diff
File: src/adapters/CurveTricryptoAdapter.sol

    /// @notice x token Ocean ID.
-   uint256 public immutable xToken;
+   uint256 internal immutable xToken;


    /// @notice y token Ocean ID.
-   uint256 public immutable yToken;
+   uint256 internal immutable yToken;
```

| Description                                      | Before      | After       | Savings          |
|--------------------------------------------------|-------------|-------------|---------------------|
| CurveTricryptoAdapter.sol AVG Deployment Costs   | 1,882,429   | 1,854,572   | **-27,857 (-1.48%)**    |
| CurveTricryptoAdapter.sol AVG Deployment Size    | 9,385       | 9,232       | **-153 (-1.63%)**       |

### OceanAdapter.sol

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L19

```solidity
File: src/adapters/OceanAdapter.sol

    address public immutable ocean;
```

```diff
File: src/adapters/OceanAdapter.sol

-   address public immutable ocean;
+   address public immutable ocean;
```

| Description                                       | Before      | After       | Savings          |
|---------------------------------------------------|-------------|-------------|---------------------|
| Curve2PoolAdapter.sol.sol AVG Deployment Costs    | 1,291,528   | 1,278,300   | **-13,228 (-1.02%)**    |
| Curve2PoolAdapter.sol.sol AVG Deployment Size     | 6,497       | 6,423       | **-74 (-1.14%)**        |
| CurveTricryptoAdapter.sol.sol AVG Deployment Costs| 1,882,429   | 1,869,800   | **-12,629 (-0.67%)**    |
| CurveTricryptoAdapter.sol.sol AVG Deployment Size | 9,385       | 9,315       | **-70 (-0.75%)**        |

## G3 - Refactor onlyOcean modifier to save gas

Optimizing the `onlyOcean` modifier in the `OceanAdapter.sol` contract to utilize conditional branching for gas efficiency.

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L37-L41

```solidity
File: src/adapters/OceanAdapter.sol

    /// @notice only allow the Ocean to call a method
    modifier onlyOcean() {
        require(msg.sender == ocean);
        _;
    }
```

```diff
File: src/adapters/OceanAdapter.sol

-   modifier onlyOcean() {
-       require(msg.sender == ocean);
-       _;
-   }

+   modifier onlyOcean() {
+       if(msg.sender == ocean){_;} else {revert();}
+   }
```

| Description                                   | Before    | After     | Savings        |
|-----------------------------------------------|-----------|-----------|-------------------|
| Curve2PoolAdapter.sol AVG Deployment Costs    | 1,291,528 | 1,290,528 | **-1,000 (-0.08%)**   |
| Curve2PoolAdapter.sol AVG Deployment Size     | 6,497     | 6,492     | **-5 (-0.08%)**       |