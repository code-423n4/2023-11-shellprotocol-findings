
# Tokens with decimals larger than 18 decimals may break the contract 
## Instances
* [ Curve2PoolAdapter.sol #82](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L82)
* [ Curve2PoolAdapter.sol #89](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L89) 


# Validate tokenAddress before use
## Instances
[Ocean.sol #821](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L821)

# Check if amount > feeCharged to avoid fees greater than the amount
* [Ocean.sol #867](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L867)
* [Ocean.sol #966](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L966) 

# Emit error on else statement
The function executes logic on if statement but does not emit an error if condition is not true and logic is not executed.
## Instances
* [Ocean.sol #1167](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L1167)
* [Ocean.sol #1043](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L1043)
* [Ocean.sol #1009](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L1009)
## Mitigation
* Emit some sort of error that shows that the condition required is not true and the function logic is not executed.

# Missing parameter in error message
## Instances
* [Ocean.sol #878](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L878) Add `tokenAddress` to error message 







# Remove and modify redundant code
The code checks for the same three conditions twice in the same function and does logic A in the first 3 checks, then does the same checks from before and does logic B separately, combine logic A and logic B  to the first 3 if statements respectively for cleaner and less redundant code.
## Instances
[Curve2PoolAdapter.sol #177-183](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L177C9-L183C10)
```js
-162 if (action == ComputeType.Swap) {
           //logic
        } else if (action == ComputeType.Deposit) {
           //logic
        } else {
    //logic
        }
```

```js
-177 if (action == ComputeType.Swap) {
            emit Swap(inputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);
        } else if (action == ComputeType.Deposit) {
            emit Deposit(inputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);
        } else {
            emit Withdraw(outputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);
        }
```
## Mitigation

Move the emits to the above if statements
[Curve2PoolAdapter.sol  #162-171](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L162) 







# Bulky functions need to be cleaned
## Instances
* [Ocean.sol #597](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L597)

## Mitigation
Divide the function into smaller functions rather than a lot of if statements with big bodies this way follows separation of concerns and more cleaner and readable code


# General
* ### Contracts are not using their OZ upgradeable counterpart
* ### Emit messages on token wrapping and un-wrapping
* ### Protocol is not pausable