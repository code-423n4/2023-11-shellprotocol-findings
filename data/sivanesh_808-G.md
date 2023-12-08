

| S.No | Title                                                                                        |
|------|----------------------------------------------------------------------------------------------|
| G-01 | Optimizing Bitwise Operations in Solidity for Gas Efficiency                                  |
| G-02 | Inlining Constant Return Values in Functions                                                  |
| G-03 | Use Short-Circuit Evaluation in Conditional Statements                                        |
| G-04 | Optimizing Gas Consumption in `computeOutputAmount` Through Unwrap Fee Divisor Caching        |
| G-05 | Optimization of `_convertDecimals` for Gas Efficiency Using Precomputed Constants             |
| G-06 | Optimization of Struct Handling in `wrapToken` and `unwrapToken` Functions                    |
| G-07 | Simplify `wrapToken` Function                                                                 |
| G-08 | Efficient Token Approval Mechanism in DeFi Contracts                                          |
| G-09 | Efficient Division Handling in `computeOutputAmount` Using Assembly                          |
| G-10 | Optimizing Storage Reads in `_fetchInteractionId` Using Assembly                              |
| G-11 | Assembly for Interaction Type Calculation                                                     |
| G-12 | Optimizing Memory Usage in the `unwrapToken` Function                                         |
| G-13 | Optimizing Gas Usage in the `primitiveOutputAmount` Function with Inline Assembly             |
| G-14 | Optimization of `_calculateUnwrapFee` Function                                                |
| G-15 | Enhancing `_grantFeeToOcean` Through Inline Assembly                                          |
| G-16 | Gas Efficiency Improvement for the `_decreaseBalanceOfPrimitive` Function                     |



## [G-01] Optimizing Bitwise Operations in Solidity for Gas Efficiency

### Description:
Efficient data handling and manipulation are crucial for optimizing gas consumption. A common scenario involves packing multiple values into a single `bytes32` type. The original implementation of `_fetchInteractionId` uses `abi.encodePacked` for packing, which, while flexible, is less gas-efficient for simple types. The optimized version leverages direct bitwise operations, offering a more gas-efficient approach.

### Original Code Snippet:
```solidity
function _fetchInteractionId(address token, uint256 interactionType) internal pure returns (bytes32) {
    uint256 packedValue = uint256(uint160(token));
    packedValue |= interactionType << 248;
    return bytes32(abi.encodePacked(packedValue));
}
```
In this original implementation, the address is first cast to `uint160`, then to `uint256`, and combined with the `interactionType` shifted 248 bits to the left. Finally, `abi.encodePacked` is used to pack these values into a `bytes32`.

### Optimized Code Snippet:
```solidity
function _fetchInteractionIdOptimized(address token, uint256 interactionType) internal pure returns (bytes32) {
    return bytes32(uint256(uint160(token)) | (interactionType << 248));
}
```
The optimized version simplifies the process by performing a direct bitwise operation. The `interactionType` is shifted 248 bits to the left and combined with the address cast to `uint256`. This approach eliminates the need for `abi.encodePacked`, resulting in reduced gas consumption.


### Link : [OceanAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L99)


## [G-02] Inlining Constant Return Values in Functions

#### Detailed Description:
Functions like `getTokenSupply` in the provided contract always return a constant value (in this case, `0`). These constant return functions can be optimized by directly using their return values where they are called, instead of making function calls. This approach saves gas by avoiding unnecessary function call overhead.

### Actual Code:
```solidity
function getTokenSupply(uint256 tokenId) external view override returns (uint256) {
    return 0;
}
```

### Optimized Approach:
Instead of calling `getTokenSupply`, replace the function call with its constant return value `0` directly in the code. For example, if you have a line of code like this:

```solidity
uint256 supply = getTokenSupply(someTokenId);
```

You can replace it with:

```solidity
uint256 supply = 0;
```

### Link : [OceanAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L124)

## [G-03] Use Short-Circuit Evaluation in Conditional Statements

### Description:
Short-circuit evaluation is a programming technique used in evaluating expressions with logical operators such as `&&` (logical AND) and `||` (logical OR). In short-circuit evaluation, the second argument is only executed or evaluated if the first argument does not suffice to determine the value of the expression. This means when using `&&`, if the first operand is false, the entire expression must be false, and thus, there's no need to evaluate the second operand. Similarly, for `||`, if the first operand is true, the entire expression must be true, and again, there's no need to evaluate the second operand.

In the context of Solidity and smart contracts, using short-circuit evaluation can save gas because it avoids unnecessary computation. This is particularly beneficial in scenarios where the first condition is often sufficient to determine the outcome, and the second condition involves more expensive operations such as external function calls or state variable reads.

### Original Code:
In the provided snippet, the condition checks two things: whether the token is not of the primitive and if the input amount is greater than 0.
```solidity
if (_isNotTokenOfPrimitive(inputToken, primitive) && (inputAmount > 0)) {
    _mintWithoutSafeTransferAcceptanceCheck(primitive, inputToken, inputAmount);
}
```
The function `_isNotTokenOfPrimitive` likely involves some computation or state checking, which can be more expensive than a simple numerical comparison.

### Optimized Code:
By rearranging the conditions, we can take advantage of short-circuit evaluation. The cheaper operation (in this case, `inputAmount > 0`) is evaluated first. If this condition is false, the second condition (`_isNotTokenOfPrimitive(inputToken, primitive)`) is not evaluated, saving gas.
```solidity
if (inputAmount > 0 && _isNotTokenOfPrimitive(inputToken, primitive)) {
    _mintWithoutSafeTransferAcceptanceCheck(primitive, inputToken, inputAmount);
}
```

### Gas Saving Explanation:
In this optimized code, if `inputAmount` is zero, the EVM skips the evaluation of `_isNotTokenOfPrimitive`. This is effective because:

### Link : [Ocean.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L1009)


## [G-04] Optimizing Gas Consumption in `computeOutputAmount` Through Unwrap Fee Divisor Caching

### Description:
The original implementation of the `computeOutputAmount` function in the `OceanAdapter` contract makes an external call to `IOceanInteractions(ocean).unwrapFeeDivisor()` on each execution. This external call is gas-intensive and can be optimized. The optimization involves caching the value of `unwrapFeeDivisor` in a state variable during contract initialization. This reduces the need for repetitive external calls, thus saving gas. This approach is particularly effective if the `unwrapFeeDivisor` value does not change frequently.

### Original Code:
```solidity
function computeOutputAmount(
    uint256 inputToken,
    uint256 outputToken,
    uint256 inputAmount,
    address,
    bytes32 metadata
)
    external
    view
    returns (uint256 outputAmount)
{
    uint256 unwrapFee = inputAmount / IOceanInteractions(ocean).unwrapFeeDivisor();
    uint256 unwrappedAmount = inputAmount - unwrapFee;

    // Placeholder for actual business logic
    outputAmount = unwrappedAmount + uint256(metadata);
}
```

### Optimized Code:
```solidity


    function computeOutputAmountOptimized(
        uint256 inputToken,
        uint256 outputToken,
        uint256 inputAmount,
        address,
        bytes32 metadata
    )
        external
        view
        returns (uint256 outputAmount)
    {
        uint256 unwrapFee = inputAmount / unwrapFeeDivisorCache;
        uint256 unwrappedAmount = inputAmount - unwrapFee;

        // Placeholder for actual business logic
        outputAmount = unwrappedAmount + uint256(metadata);
    }

```

### Link : [OceanAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L55)


## [G-05]Optimization of `_convertDecimals` for Gas Efficiency Using Precomputed Constants**

### Description:
The `_convertDecimals` function in the `OceanAdapter` contract is designed for converting token amounts between different decimal bases. While the original implementation effectively handles this conversion, it performs exponentiation for each operation, which can be gas-intensive. The optimization strategy focuses on commonly used decimal conversions (e.g., between 6 and 18 decimals) and utilizes precomputed constants to replace the exponentiation operations. This approach is expected to reduce gas consumption for these specific scenarios while maintaining the original functionality for other cases.

### Original Code Snippet:
```solidity
function _convertDecimalsOriginal(
    uint8 decimalsFrom,
    uint8 decimalsTo,
    uint256 amountToConvert
)
    public
    pure
    returns (uint256 convertedAmount)
{
    if (decimalsFrom == decimalsTo) {
        convertedAmount = amountToConvert;
    } else if (decimalsFrom < decimalsTo) {
        uint256 shift = 10 ** (uint256(decimalsTo - decimalsFrom));
        convertedAmount = amountToConvert * shift;
    } else {
        uint256 shift = 10 ** (uint256(decimalsFrom - decimalsTo));
        convertedAmount = amountToConvert / shift;
    }
}
```

### Optimized Code:
```solidity
function _convertDecimalsOptimized(
    uint8 decimalsFrom,
    uint8 decimalsTo,
    uint256 amountToConvert
)
    public
    pure
    returns (uint256 convertedAmount)
{
    if (decimalsFrom == decimalsTo) {
        return amountToConvert;
    } else if (decimalsFrom == 6 && decimalsTo == 18) {
        return amountToConvert * 1e12; // Precomputed 10 ** 12
    } else if (decimalsFrom == 18 && decimalsTo == 6) {
        return amountToConvert / 1e12; // Precomputed 10 ** 12
    } else if (decimalsFrom < decimalsTo) {
        return amountToConvert * 10 ** (decimalsTo - decimalsFrom);
    } else {
        return amountToConvert / 10 ** (decimalsFrom - decimalsTo);
    }
}
```
### Link : [OceanAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L138)

## [G-06] Optimization of Struct Handling in `wrapToken` and `unwrapToken` Functions**

### Description:
The original implementation of the `wrapToken` and `unwrapToken` functions in the `Curve2PoolAdapter` contract involves creating new `Interaction` memory structs for every function call. This method, while straightforward, entails additional memory allocation, which increases gas consumption. The optimization focuses on streamlining memory usage by directly passing struct parameters to the `doInteraction` function, thus bypassing the need for explicit struct creation. This modification aims to minimize memory operations, potentially leading to gas savings.

### Original `wrapToken` and `unwrapToken` Functions:
```solidity
function wrapTokenOriginal(uint256 tokenId, uint256 amount) internal {
    Interaction memory interaction = Interaction({
        interactionTypeAndAddress: bytes32(0), // Simplified for testing
        inputToken: 0,
        outputToken: 0,
        specifiedAmount: amount,
        metadata: bytes32(0)
    });
    doInteraction(interaction);
}

function unwrapTokenOriginal(uint256 tokenId, uint256 amount) internal {
    Interaction memory interaction = Interaction({
        interactionTypeAndAddress: bytes32(0), // Simplified for testing
        inputToken: 0,
        outputToken: 0,
        specifiedAmount: amount,
        metadata: bytes32(0)
    });
    doInteraction(interaction);
}


```

### Optimized `wrapToken` and `unwrapToken` Functions (Option 1):
```solidity
function wrapTokenOptimized(uint256 tokenId, uint256 amount) internal {
    doInteraction(Interaction({
        interactionTypeAndAddress: bytes32(0), // Simplified for testing
        inputToken: 0,
        outputToken: 0,
        specifiedAmount: amount,
        metadata: bytes32(0)
    }));
}

function unwrapTokenOptimized(uint256 tokenId, uint256 amount) internal {
    doInteraction(Interaction({
        interactionTypeAndAddress: bytes32(0), // Simplified for testing
        inputToken: 0,
        outputToken: 0,
        specifiedAmount: amount,
        metadata: bytes32(0)
    }));
}
```
### Link : [Curve2PoolAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L102)



## [G-07] Simplify `wrapToken` Function

### Description
In the `wrapToken` function, there's a conditional check to differentiate the handling of `zToken` from other tokens. This can be streamlined by unifying the creation of the `Interaction` struct and reducing the number of conditional branches.

### Original Code
```solidity
function wrapToken(uint256 tokenId, uint256 amount) internal override {
    Interaction memory interaction;

    if (tokenId == zToken) {
        interaction = Interaction({
            interactionTypeAndAddress: 0,
            inputToken: 0,
            outputToken: 0,
            specifiedAmount: 0,
            metadata: bytes32(0)
        });
        IOceanInteractions(ocean).doInteraction{ value: amount }(interaction);
    } else {
        interaction = Interaction({
            interactionTypeAndAddress: _fetchInteractionId(underlying[tokenId], uint256(InteractionType.WrapErc20)),
            inputToken: 0,
            outputToken: 0,
            specifiedAmount: amount,
            metadata: bytes32(0)
        });
        IOceanInteractions(ocean).doInteraction(interaction);
    }
}
```

### Optimized Code
```solidity
function wrapToken(uint256 tokenId, uint256 amount) internal override {
    uint256 interactionType = (tokenId == zToken) ? 0 : uint256(InteractionType.WrapErc20);
    address interactionAddress = (tokenId == zToken) ? address(0) : underlying[tokenId];

    Interaction memory interaction = Interaction({
        interactionTypeAndAddress: _fetchInteractionId(interactionAddress, interactionType),
        inputToken: 0,
        outputToken: 0,
        specifiedAmount: amount,
        metadata: bytes32(0)
    });

    if (tokenId == zToken) {
        IOceanInteractions(ocean).doInteraction{ value: amount }(interaction);
    } else {
        IOceanInteractions(ocean).doInteraction(interaction);
    }
}
```

In the optimized version:
- The creation of the `Interaction` struct is unified for both `zToken` and other tokens, reducing the code duplication.
- The use of a ternary operator simplifies the determination of `interactionType` and `interactionAddress`.
- The conditional check for `zToken` is maintained only for the `doInteraction` call, where the distinction between ETH and ERC20 tokens is necessary.



### Link : [Curve2PoolAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L118)

## [G-08] Efficient Token Approval Mechanism in DeFi Contracts**

### Description:
The `_approveToken` function in the `CurveTricryptoAdapter` contract is integral for setting allowance for ERC20 tokens to be spent by external contracts. The original implementation indiscriminately sets the maximum allowance without checking the existing allowance. This approach, while ensuring sufficient allowance for token operations, can lead to unnecessary and costly state changes on the blockchain. The optimization involves introducing a conditional check to determine if the current allowance is already set to the desired amount, thereby avoiding redundant `approve` transactions and saving gas.

### Original `_approveToken` Function:
```solidity
function _approveToken(address tokenAddress) private {
    IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);
    IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);
}
```

### Optimized `_approveToken` Function:
```solidity
function _approveToken(address tokenAddress) private {
    IERC20Metadata token = IERC20Metadata(tokenAddress);
    uint256 maxAllowance = type(uint256).max;

    if (token.allowance(address(this), ocean) < maxAllowance) {
        token.approve(ocean, maxAllowance);
    }

    if (token.allowance(address(this), primitive) < maxAllowance) {
        token.approve(primitive, maxAllowance);
    }
}
```

### Technical Explanation:
- **State Change Cost**: In Ethereum, state changes (like updating storage variables) are costly operations in terms of gas. The `approve` function call results in a state change on the blockchain. By reducing the frequency of these calls, significant gas savings can be achieved.
- **Read vs. Write Operations**: Reading data from the blockchain (like checking the current allowance with `allowance()`) is less expensive than writing data (like setting a new allowance with `approve()`). The optimized function leverages this by reading the current allowance and conditionally executing the `approve()` function.
- **Idempotency**: The optimization ensures the idempotency of the `_approveToken` function. It means that repeated calls to this function with the same parameters do not change the state unnecessarily and, thus, do not incur additional gas costs.
- **Security Considerations**: While the optimization adds a conditional check, it does not compromise the security of the token approval process. The function still ensures that sufficient allowance is set for the Ocean and Curve pool contracts to operate with the token.



#### Link : [CurveTricryptoAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L241)




##  [G-09] Efficient Division Handling in `computeOutputAmount` Using Assembly

### Description
The `computeOutputAmount` function involves a division operation for calculating the `unwrapFee`. This can be optimized using assembly, particularly when the divisor is a constant, allowing for more direct manipulation of the EVM's arithmetic capabilities.

### Original Code
```solidity
uint256 unwrapFee = inputAmount / IOceanInteractions(ocean).unwrapFeeDivisor();
```

### Optimized Code
```solidity
assembly {
    let divisor := sload(IOceanInteractions(ocean).unwrapFeeDivisor.slot)
    unwrapFee := div(inputAmount, divisor)
}
```

### Link : [OceanAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L70)


##  [G-10] Optimizing Storage Reads in `_fetchInteractionId` Using Assembly

### Description
The `_fetchInteractionId` function can be optimized by directly reading from storage using assembly. This is particularly effective when dealing with simple storage operations, as it bypasses the overhead of Solidity's automatic getter generation.

### Original Code
```solidity
packedValue |= interactionType << 248;
```

### Optimized Code
```solidity
assembly {
    let packedValue := sload(token_slot)
    packedValue := or(packedValue, shl(248, interactionType))
    sstore(token_slot, packedValue)
}
```
**Note:** `token_slot` should be replaced with the actual storage slot of the variable in context. The `shl` operation shifts `interactionType` 248 bits to the left, equivalent to the original `<<` operation in Solidity.


### Link : [OceanAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L101)



## [G-11] Assembly for Interaction Type Calculation:
   In the `_determineComputeType` function, the computation to determine the interaction type can potentially be optimized using assembly, especially if it's a hotspot in terms of gas usage.

   ### Original Code:
   ```solidity
   if (((inputToken == xToken) && (outputToken == yToken)) || ((inputToken == yToken) && (outputToken == xToken))) {
       return ComputeType.Swap;
   } else if (((inputToken == xToken) || (inputToken == yToken)) && (outputToken == lpTokenId)) {
       return ComputeType.Deposit;
   } else if ((inputToken == lpTokenId) && ((outputToken == xToken) || (outputToken == yToken))) {
       return ComputeType.Withdraw;
   } else {
       revert INVALID_COMPUTE_TYPE();
   }
   ```

   ### Optimized Code
   ```solidity
   assembly {
       let swapCondition := or(and(eq(inputToken, sload(xToken_slot)), eq(outputToken, sload(yToken_slot))), and(eq(inputToken, sload(yToken_slot)), eq(outputToken, sload(xToken_slot))))
       let depositCondition := and(or(eq(inputToken, sload(xToken_slot)), eq(inputToken, sload(yToken_slot))), eq(outputToken, sload(lpTokenId_slot)))
       let withdrawCondition := and(eq(inputToken, sload(lpTokenId_slot)), or(eq(outputToken, sload(xToken_slot)), eq(outputToken, sload(yToken_slot))))
       
       switch swapCondition
       case 1 {
           mstore(0x00, Swap)
           return(0x00, 0x20)
       }
       switch depositCondition
       case 1 {
           mstore(0x00, Deposit)
           return(0x00, 0x20)
       }
       switch withdrawCondition
       case 1 {
           mstore(0x00, Withdraw)
           return(0x00, 0x20)
       }
       revert(0x00, 0x00)
   }
   ```

#### Link : [Curve2PoolAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L201)


## [G-12] Optimizing Memory Usage in the `unwrapToken` Function

### Description: 
This optimization aims to streamline memory usage in the `unwrapToken` function of the Solidity contract. In the original code, a new instance of the `Interaction` struct is created in memory for each call of the function. This process involves multiple memory allocations which can be optimized using inline assembly. Inline assembly allows for more direct and efficient management of memory by manually assigning values to specific memory slots, thereby potentially reducing gas costs associated with memory allocation and automatic struct packing/unpacking in Solidity.

### Original Full Code:
```solidity
function unwrapToken(uint256 tokenId, uint256 amount) internal override {
    Interaction memory interaction;

    if (tokenId == zToken) {
        interaction = Interaction({
            interactionTypeAndAddress: _fetchInteractionId(address(0), uint256(InteractionType.UnwrapEther)),
            inputToken: 0,
            outputToken: 0,
            specifiedAmount: amount,
            metadata: bytes32(0)
        });
    } else {
        interaction = Interaction({
            interactionTypeAndAddress: _fetchInteractionId(underlying[tokenId], uint256(InteractionType.UnwrapErc20)),
            inputToken: 0,
            outputToken: 0,
            specifiedAmount: amount,
            metadata: bytes32(0)
        });
    }

    IOceanInteractions(ocean).doInteraction(interaction);
}
```

### Optimized Full Code:
```solidity
function unwrapToken(uint256 tokenId, uint256 amount) internal override {
    // Assuming interactionTypeAndAddress is at slot 0 of the struct
    // and each subsequent field is at the next sequential 32-byte slot.
    bytes32 interactionTypeAndAddress = (tokenId == zToken) ?
        _fetchInteractionId(address(0), uint256(InteractionType.UnwrapEther)) :
        _fetchInteractionId(underlying[tokenId], uint256(InteractionType.UnwrapErc20));

    assembly {
        let interaction := mload(0x40) // Load the free memory pointer
        mstore(interaction, interactionTypeAndAddress) // Store interactionTypeAndAddress at the start
        mstore(add(interaction, 0x20), 0) // inputToken at the next 32-byte slot
        mstore(add(interaction, 0x40), 0) // outputToken at the next 32-byte slot
        mstore(add(interaction, 0x60), amount) // specifiedAmount at the next 32-byte slot
        mstore(add(interaction, 0x80), 0) // metadata (empty bytes32) at the next 32-byte slot
        mstore(0x40, add(interaction, 0xA0)) // Update the free memory pointer

        let success := call(
            gas(), // Send all available gas
            ocean, // IOceanInteractions contract address
            0, // No Ether to be sent
            interaction, // Pointer to the start of the interaction data in memory
            0xA0, // Size of the interaction data (5 slots * 32 bytes each)
            0, // Output data ignored
            0  // Output data size
        )

        // Revert if the call was not successful
        if iszero(success) {
            revert(0, 0)
        }
    }
}
```

### Explanation of the Optimized Code:
- The `unwrapToken` function is optimized to use inline assembly for creating and passing the `Interaction` struct to the `IOceanInteractions(ocean).doInteraction` function call.
- We first calculate the `interactionTypeAndAddress` outside of assembly based on the `tokenId`.
- Inside the assembly block, we start by loading the current free memory pointer (`mload(0x40)`).
- We then sequentially store each field of the `Interaction` struct in memory. This is manually done for each field (`interactionTypeAndAddress`, `inputToken`, `outputToken`, `specifiedAmount`, and `metadata`) at specific memory offsets.
- After storing all struct fields, we update the free memory pointer to point after the last field of the struct.
- Finally, a low-level `call` is made to the `IOceanInteractions(ocean).doInteraction` function, passing in the pointer to the struct in memory and the size of the data.
- The call's success is checked, and the function reverts if the call was unsuccessful.


### Link : [CurveTricryptoAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L147)

## [G-13] Optimizing Gas Usage in the `primitiveOutputAmount` Function with Inline Assembly

### Description:
The `primitiveOutputAmount` function in the given Solidity contract can be optimized for gas usage by reducing redundant storage read operations. In the original version, the function accesses the `decimals` and `indexOf` mappings multiple times, each requiring a separate SLOAD operation, which is costly in terms of gas. By using inline assembly, we can directly read these values from storage, thereby reducing the number of SLOAD operations and potentially lowering the gas cost.

### Original Full Code:
```solidity
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
    uint256 rawInputAmount = _convertDecimals(NORMALIZED_DECIMALS, decimals[inputToken], inputAmount);

    ComputeType action = _determineComputeType(inputToken, outputToken);

    uint256 _balanceBefore = _getBalance(underlying[outputToken]);

    // avoid multiple SLOADS
    uint256 indexOfInputAmount = indexOf[inputToken];
    uint256 indexOfOutputAmount = indexOf[outputToken];

    // Rest of the function implementation...
}
```

### Optimized Full Code:
```solidity
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
    uint256 rawInputAmount;
    uint256 indexOfInputAmount;
    uint256 indexOfOutputAmount;
    uint8 decimalInputToken;

    // Inline assembly for optimized storage access
    assembly {
        // Calculate the storage position for decimals[inputToken]
        let pos := keccak256(add(inputToken, 1), 1) // Assuming the slot for the `decimals` mapping is 1
        decimalInputToken := sload(pos)

        // Calculate the storage positions for indexOf[inputToken] and indexOf[outputToken]
        pos := keccak256(add(inputToken, 1), 2) // Assuming the slot for the `indexOf` mapping is 2
        indexOfInputAmount := sload(pos)

        pos := keccak256(add(outputToken, 1), 2)
        indexOfOutputAmount := sload(pos)
    }

    rawInputAmount = _convertDecimals(NORMALIZED_DECIMALS, decimalInputToken, inputAmount);
    ComputeType action = _determineComputeType(inputToken, outputToken);

    uint256 _balanceBefore = _getBalance(underlying[outputToken]);

    // The rest of the function implementation remains unchanged...
}
```

### Explanation of the Optimized Code:
- In the optimized version, we introduce an inline assembly block to replace the direct Solidity mapping access for `decimals[inputToken]`, `indexOf[inputToken]`, and `indexOf[outputToken]`.
- The `keccak256` hash function is used within assembly to calculate the storage positions of these values. This requires understanding the storage layout of Solidity, where mapping values are stored at the hash of the key and the slot number.
- We use `sload` to directly load the values from these calculated storage positions into local variables (`decimalInputToken`, `indexOfInputAmount`, `indexOfOutputAmount`).
- These values are then used in the subsequent calculations and logic of the function, replacing the original mapping accesses.
- The rest of the function remains unchanged, with the optimized part focusing solely on the storage read optimizations.

### Link : [CurveTricryptoAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L178)



## [G-13] Optimizing Gas Usage in `_getBalance` Function

### Description:
The `_getBalance` function in the Solidity contract performs a balance check, which can be optimized for gas usage. The original code handles two different scenarios based on the token address: querying the contract's Ether balance and querying an ERC20 token balance. This check can be streamlined using inline assembly to directly interact with the balance (for Ether) and to call the balanceOf function (for ERC20 tokens), thereby reducing the overhead associated with high-level Solidity constructs.

### Original Full Code:
```solidity
function _getBalance(address tokenAddress) internal view returns (uint256 balance) {
    if (tokenAddress == underlying[zToken]) {
        return address(this).balance;
    } else {
        return IERC20Metadata(tokenAddress).balanceOf(address(this));
    }
}
```

### Optimized Full Code:
```solidity
function _getBalance(address tokenAddress) internal view returns (uint256 balance) {
    assembly {
        switch eq(tokenAddress, sload(zToken_slot)) // Compare with zToken; assuming zToken_slot is the storage slot of zToken
        case 1 {
            // Handle Ether balance
            balance := selfbalance() // Use selfbalance for optimized gas usage
        }
        default {
            // Handle ERC20 token balance
            let ptr := mload(0x40) // Free memory pointer
            mstore(ptr, 0x70a0823100000000000000000000000000000000000000000000000000000000) // balanceOf signature
            mstore(add(ptr, 0x04), address()) // Append address to the balanceOf call

            // Perform the call to the ERC20 token contract
            if iszero(staticcall(gas(), tokenAddress, ptr, 0x24, ptr, 0x20)) {
                revert(0, 0)
            }

            balance := mload(ptr) // Load the returned balance
        }
    }
}
```

### Explanation of the Optimized Code:
- The optimized code uses an inline assembly block to handle the balance check.
- We use a `switch` statement to differentiate between the Ether balance check and the ERC20 token balance check.
- For the Ether balance, `selfbalance` is used instead of `address(this).balance`, which is a more gas-efficient way to get the contract's Ether balance in assembly.
- For the ERC20 token balance, we manually construct the calldata for the `balanceOf` function, including its signature and the address argument.
- A `staticcall` is then used to query the balance, which is a low-level call that doesn't modify the state, suitable for view functions like `balanceOf`.
- If the call is successful, the balance is loaded from memory and returned. If not, the function reverts.


### Link : [CurveTricryptoAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L249)



## [G-14] Optimization of `_calculateUnwrapFee` Function



### Detailed Description
The `_calculateUnwrapFee` function is a straightforward computation involving division. In Solidity, division is typically compiled to the EVM `DIV` opcode. By employing inline assembly, we aim to directly utilize EVM opcodes, potentially reducing overhead introduced by high-level Solidity constructs. The optimization focuses on directly accessing storage and performing the division operation within the assembly block. However, it's important to note that the gas savings may be marginal due to the simplicity of the operation.

###  Original Code:

```solidity
function _calculateUnwrapFee(uint256 unwrapAmount) private view returns (uint256 feeCharged) {
    feeCharged = unwrapAmount / unwrapFeeDivisor;
}
```

 ###  Optimized Code

```solidity
function _calculateUnwrapFee(uint256 unwrapAmount) private view returns (uint256 feeCharged) {
    assembly {
        feeCharged := div(unwrapAmount, sload(unwrapFeeDivisor_slot)) // unwrapFeeDivisor_slot is the storage slot of unwrapFeeDivisor
    }
}
```

## Link : [Ocean.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L1158)

## [G-15] Enhancing `_grantFeeToOcean` Through Inline Assembly

### Detailed Description
The `_grantFeeToOcean` function includes a conditional check and a function call to `_mintWithoutSafeTransferAcceptanceCheck`. This optimization aims to streamline the conditional check using assembly language. However, as the bulk of the gas cost likely comes from the function call itself, the optimization potential may be limited. The assembly block manually checks the condition and performs the function call if necessary.

### Original Code:

```solidity
function _grantFeeToOcean(uint256 oceanId, uint256 amount) private {
    if (amount > 0) {
        _mintWithoutSafeTransferAcceptanceCheck(owner(), oceanId, amount);
    }
}
```

### Optimized Code

```solidity
function _grantFeeToOcean(uint256 oceanId, uint256 amount) private {
    assembly {
        if gt(amount, 0) {
            // Prepare function call to _mintWithoutSafeTransferAcceptanceCheck
            let result := call(gas(), sload(_mintWithoutSafeTransferAcceptanceCheck_address_slot),
                              0, add(data, 32), msize(), 0, 0)
            // Handle result (if necessary)
        }
    }
}
```

### Link : [Ocean.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L1166)


## [G-16] Gas Efficiency Improvement for the `_decreaseBalanceOfPrimitive` Function

### Detailed Description:

The objective is to optimize the `_decreaseBalanceOfPrimitive` function for gas efficiency without using inline assembly. This function checks whether a given token is not associated with a provided primitive contract and, if not, burns a specified amount of the token from the primitive’s balance. The optimization focuses on reducing unnecessary computation and simplifying the conditional logic to potentially lower gas costs.

###  Original Code:

```solidity
function _decreaseBalanceOfPrimitive(address primitive, uint256 outputToken, uint256 outputAmount) internal {
    if (_isNotTokenOfPrimitive(outputToken, primitive) && (outputAmount > 0)) {
        _burn(primitive, outputToken, outputAmount);
    }
}
```

###  Optimized Code:

```solidity
function _decreaseBalanceOfPrimitive(address primitive, uint256 outputToken, uint256 outputAmount) internal {
    if (outputAmount == 0) return;
    if (_isNotTokenOfPrimitive(outputToken, primitive)) {
        _burn(primitive, outputToken, outputAmount);
    }
}
```

### Explanation of Optimization:

1. **Early Return for Zero Amounts:** The function now checks first if `outputAmount` is zero and, if so, returns immediately. This optimizes the function by avoiding the more expensive operation of checking whether the token belongs to the primitive when it's unnecessary.

2. **Simplified Conditional Logic:** The original conditional statement combined two checks in one line. By separating these into two if-statements, the code becomes clearer, and there's potential for gas savings when `outputAmount` is frequently zero.

3. **Maintaining Readability and Maintainability:** The changes ensure that the code remains easy to understand and maintain, crucial for the long-term health of the codebase.



### Link : [Ocean.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L1038)


   
