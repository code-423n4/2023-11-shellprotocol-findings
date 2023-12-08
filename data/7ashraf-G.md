
# Using both named returns and a return statement isn’t necessary
## Instances
* [ocean.sol #217](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L217)
* [ocean.sol #244](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L244)
* [ocean.sol #267](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L267)  

# Loops can be implemented more efficiently 
## Instances

* [ocean.sol 463](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L463)
* [ocean.sol 501](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L501)
### Recommended implementation:

```

uint length = arr.length;

for (uint i; i < length; i++) {

//Operations not affecting the length of the array.

}
```

# Surround by unchecked 
safe mathematical operations can be surrounded by unchecked to save gas
## Instances

[ocean.sol 867](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L867) 
[ocean.sol 966](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L966) 
[ocean.sol 1143](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L1143)

# unrequired 3 if statements blocks, remove to save gas and modify code
The code checks for the same three conditions twice in the same function and does logic A in the first 3 checks, then does the same checks from before and does logic B separately, combine logic A and logic B  to the first 3 if statements respectively and save gas
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
[Curve2PoolAdapter.sol  #162-171](.) 

