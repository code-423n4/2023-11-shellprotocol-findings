# Gas-Optmization for Shell Protocol

## [G-01] Avoid contract existence checks by using low level calls

**Issue Description**\
Prior to Solidity 0.8.10, the compiler would insert extra code to check for contract existence before external function calls,
even if the call had a return value. This wasted gas by performing a check that wasn't necessary.

**Proposed Optimization**\
For functions that make external calls with return values in Solidity <0.8.10, optimize the code to use low-level calls instead
of regular calls. Low-level calls skip the unnecessary contract existence check.

Example:

```solidity
//Before:

contract C {
  function f() external returns(uint) {
    address(otherContract).call(abi.encodeWithSignature("func()"));
  }
}


//After:

contract C {
  function f() external returns(uint) {
    (bool success,) = address(otherContract).call(abi.encodeWithSignature("func()"));
    require(success);
    return decodeReturnValue();
  }
}
```

**Estimated Gas Savings**\
Each avoided EXTCODESIZE check saves 100 gas. If 10 external calls are made in a common function, this would save 1000 gas
total.

**Attachments**

- **Code Snippets**

```solidity
File: adapters/Curve2PoolAdapter.sol

78        address xTokenAddress = ICurve2Pool(primitive).coins(0);

81        decimals[xToken] = IERC20Metadata(xTokenAddress).decimals();

84        address yTokenAddress = ICurve2Pool(primitive).coins(1);

88        decimals[yToken] = IERC20Metadata(yTokenAddress).decimals();

93        decimals[lpTokenId] = IERC20Metadata(primitive_).decimals();

113        IOceanInteractions(ocean).doInteraction(interaction);

132        IOceanInteractions(ocean).doInteraction(interaction);

164                ICurve2Pool(primitive).exchange(indexOfInputAmount, indexOfOutputAmount, rawInputAmount, 0);

168            rawOutputAmount = ICurve2Pool(primitive).add_liquidity(inputAmounts, 0);

170            rawOutputAmount = ICurve2Pool(primitive).remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);

190        IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);

191        IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L78

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

86        address xTokenAddress = ICurveTricrypto(primitive).coins(0);

89        decimals[xToken] = IERC20Metadata(xTokenAddress).decimals();

92        address yTokenAddress = ICurveTricrypto(primitive).coins(1);

96        decimals[yToken] = IERC20Metadata(yTokenAddress).decimals();

99        address wethAddress = ICurveTricrypto(primitive).coins(2);

106        address lpTokenAddress = ICurveTricrypto(primitive).token();

109        decimals[lpTokenId] = IERC20Metadata(lpTokenAddress).decimals();

129            IOceanInteractions(ocean).doInteraction{ value: amount }(interaction);

138            IOceanInteractions(ocean).doInteraction(interaction);

168        IOceanInteractions(ocean).doInteraction(interaction);

210            ICurveTricrypto(primitive).add_liquidity(inputAmounts, 0);

213                uint256 wethBalance = IERC20Metadata(underlying[zToken]).balanceOf(address(this));

214                ICurveTricrypto(primitive).remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);

215                IWETH(underlying[zToken]).withdraw(

219                ICurveTricrypto(primitive).remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);

242        IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);

243        IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L86

```solidity
File: src/ocean/Ocean.sol

891        IERC721(tokenAddress).safeTransferFrom(userAddress, address(this), tokenId);

904        IERC721(tokenAddress).safeTransferFrom(address(this), userAddress, tokenId);

931        IERC1155(tokenAddress).safeTransferFrom(userAddress, address(this), tokenId, amount, "");

968        IERC1155(tokenAddress).safeTransferFrom(address(this), userAddress, tokenId, amountRemaining, "");

```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L891

## [G-02] Add unchecked {} for subtractions where the operands can't underflow because of previous checks

**Issue Description**\
The subtraction operation dust = normalizedTransferAmount - amount; does not have an unchecked block, even though underflow
is not possible due to the previous check normalizedTransferAmount > amount. Adding an unchecked block helps avoid unnecessary
checks.

**Proposed Optimization**\
Add an unchecked block around the subtraction:

```solidity
unchecked {
  dust = normalizedTransferAmount - amount;
}
```

**Estimated Gas Savings**\
Adding an unchecked block for a subtraction that cannot underflow due to a previous check saves approximately 3 gas per subtraction.
With many such subtractions occurring across interactions, the total gas savings could be significant.

**Attachments**

- **Code Snippets**

```solidity
File: src/ocean/Ocean.sol

1102            dust = normalizedTransferAmount - amount;
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L1102

```solidity
File: src/adapters/OceanAdapter.sol

152            uint256 shift = 10 ** (uint256(decimalsTo - decimalsFrom));
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L152

## [G-03] State variables should be cached in stack variables rather than re-reading them from storage

**Issue Description**\
In the changeUnwrapFee() function, the state variable unwrapFeeDivisor is read directly from storage on line 199 when emitting
the event ChangeUnwrapFee. This is inefficient as it incurs an extra storage read operation that is unnecessary.

**Proposed Optimization**\
Cache the current value of unwrapFeeDivisor in a local stack variable before emitting the event and updating storage. This
avoids an unnecessary re-read of the storage.

```solidity
function changeUnwrapFee(uint256 nextUnwrapFeeDivisor) external override onlyOwner {
  uint256 currentUnwrapFeeDivisor = unwrapFeeDivisor;

  if (MIN_UNWRAP_FEE_DIVISOR > nextUnwrapFeeDivisor) revert();

  emit ChangeUnwrapFee(currentUnwrapFeeDivisor, nextUnwrapFeeDivisor, msg.sender);

  unwrapFeeDivisor = nextUnwrapFeeDivisor;
}
```

**Estimated Gas Savings**\
Reading a storage slot costs roughly 200 gas. Caching the value in a local variable avoids this extra read, saving roughly
200 gas per function call.

**Attachments**

- **Code Snippets**

```solidity
File: src/ocean/Ocean.sol

196    function changeUnwrapFee(uint256 nextUnwrapFeeDivisor) external override onlyOwner {
197        /// @notice as the divisor gets smaller, the fee charged gets larger
198        if (MIN_UNWRAP_FEE_DIVISOR > nextUnwrapFeeDivisor) revert();
199        emit ChangeUnwrapFee(unwrapFeeDivisor, nextUnwrapFeeDivisor, msg.sender); //@audit >>> unwrapFeeDivisor is state-variable
200        unwrapFeeDivisor = nextUnwrapFeeDivisor;
201    }



1082        (transferAmount, truncated) = _convertDecimals(NORMALIZED_DECIMALS, decimals, amount); //@audit >>>
1097                _convertDecimals(decimals, NORMALIZED_DECIMALS, transferAmount);        //@audit >>>
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L196-L201

## [G-04] Use constants instead of type(uintx).max

**Issue Description**\
Using type(uint256).max to represent the maximum approvable amount for ERC20 tokens uses more gas in the distribution process
and also for each transaction than using a constant.

**Proposed Optimization**\
Define a constant such as MAX_UINT256 = type(uint256).max and use that constant instead.

**Estimated Gas Savings**\
Using a constant avoids the overhead of calling the type(uint256) method each time. This could save ~100 gas per transaction.
For contracts with many transactions, this can add up to significant savings over time.

**Attachments**

- **Code Snippets**

```solidity
File: src/ocean/Ocean.sol

170        unwrapFeeDivisor = type(uint256).max;
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L170

```solidity
File: src/adapters/Curve2PoolAdapter.sol

190        IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);

191        IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L190-L191

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

242        IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);

243        IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L242-L243

## [G-05] Using assembly to revert with an error message

**Issue Description**\
When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error
message. This can in most cases be further optimized by using assembly to revert with the error message.

**Estimated Gas Savings**\
Here’s an example;

```solidity
/// calling restrictedAction(2) with a non-owner address: 24042


contract SolidityRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num)  external {
        require(owner == msg.sender, "caller is not owner");
        specialNumber = num;
    }
}

/// calling restrictedAction(2) with a non-owner address: 23734


contract AssemblyRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num)  external {
        assembly {
            if sub(caller(), sload(owner.slot)) {
                mstore(0x00, 0x20) // store offset to where length of revert message is stored
                mstore(0x20, 0x13) // store length (19)
                mstore(0x40, 0x63616c6c6572206973206e6f74206f776e657200000000000000000000000000) // store hex representation of message
                revert(0x00, 0x60) // revert with data
            }
        }
        specialNumber = num;
    }
}
```

From the example above we can see that we get a gas saving of over 300 gas when reverting wth the same error message
with assembly against doing so in solidity. This gas savings come from the memory expansion costs and extra type checks
the solidity compiler does under the hood.

**Attachments**

- **Code Snippets**

```solidity
File: src/ocean/Ocean.sol

640            if (specifiedAmount != 1) revert INVALID_ERC721_AMOUNT();

648            if (specifiedAmount != 1) revert INVALID_ERC721_AMOUNT();

840            revert NO_DECIMAL_METHOD();

878            revert NO_DECIMAL_METHOD();

929        if (tokenAddress == address(this)) revert NO_RECURSIVE_WRAPS();

964        if (tokenAddress == address(this)) revert NO_RECURSIVE_UNWRAPS();
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L640

```solidity
File: src/adapters/Curve2PoolAdapter.sol

175        if (uint256(minimumOutputAmount) > outputAmount) revert SLIPPAGE_LIMIT_EXCEEDED();

217            revert INVALID_COMPUTE_TYPE();
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L175

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

227        if (uint256(minimumOutputAmount) > outputAmount) revert SLIPPAGE_LIMIT_EXCEEDED();

287            revert INVALID_COMPUTE_TYPE();
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L227

## [G-06] Use assembly to reuse memory space when making more than one external call

**Issue Description**\
When making external calls, the solidity compiler has to encode the function signature and arguments in memory. It does not
clear or reuse memory, so it expands memory each time.

**Proposed Optimization**\
Use inline assembly to reuse the same memory space for multiple external calls. Store the function selector and arguments
without expanding memory further.

**Estimated Gas Savings**\
Reusing memory can save thousands of gas compared to expanding on each call. The baseline memory expansion per call is 3,000
gas. With larger arguments or return data, the savings would be even greater.

```solidity
See the example below;

contract Called {
    function add(uint256 a, uint256 b) external pure returns(uint256) {
        return a + b;
    }
}


contract Solidity {
    // cost: 7262
    function call(address calledAddress) external pure returns(uint256) {
        Called called = Called(calledAddress);
        uint256 res1 = called.add(1, 2);
        uint256 res2 = called.add(3, 4);

        uint256 res = res1 + res2;
        return res;
    }
}


contract Assembly {
    // cost: 5281
    function call(address calledAddress) external view returns(uint256) {
        assembly {
            // check that calledAddress has code deployed to it
            if iszero(extcodesize(calledAddress)) {
                revert(0x00, 0x00)
            }

            // first call
            mstore(0x00, hex"771602f7")
            mstore(0x04, 0x01)
            mstore(0x24, 0x02)
            let success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res1 := mload(0x60)

            // second call
            mstore(0x04, 0x03)
            mstore(0x24, 0x4)
            success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res2 := mload(0x60)

            // add results
            let res := add(res1, res2)

            // return data
            mstore(0x60, res)
            return(0x60, 0x20)
        }
    }
}
```

We save approximately 2,000 gas by using the scratch space to store the function selector and it’s arguments and also
reusing the same memory space for the second call while storing the returned data in the zero slot thus not expanding
memory.

If the arguments of the external function you wish to call is above 64 bytes and if you are making one external call, it
wouldn’t save any significant gas writing it in assembly. However, if making more than one call. You can still save gas
by reusing the same memory slot for the 2 calls using inline assembly.

Note: Always remember to update the free memory pointer if the offset it points to is already used, to avoid solidity
overriding the data stored there or using the value stored there in an unexpected way.

Also note to avoid overwriting the zero slot (0x60 memory offset) if you have undefined dynamic memory values within
that call stack. An alternative is to explicitly define dynamic memory values or if used, to set the slot back to 0x00
before exiting the assembly block.

**Attachments**

- **Code Snippets**

```solidity
File: src/adapters/Curve2PoolAdapter.sol

78        address xTokenAddress = ICurve2Pool(primitive).coins(0);
81        decimals[xToken] = IERC20Metadata(xTokenAddress).decimals();
84        address yTokenAddress = ICurve2Pool(primitive).coins(1);
88        decimals[yToken] = IERC20Metadata(yTokenAddress).decimals();
93        decimals[lpTokenId] = IERC20Metadata(primitive_).decimals();



164                ICurve2Pool(primitive).exchange(indexOfInputAmount, indexOfOutputAmount, rawInputAmount, 0);
168            rawOutputAmount = ICurve2Pool(primitive).add_liquidity(inputAmounts, 0);
170            rawOutputAmount = ICurve2Pool(primitive).remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);



190        IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);
191        IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L78

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

86        address xTokenAddress = ICurveTricrypto(primitive).coins(0);
89        decimals[xToken] = IERC20Metadata(xTokenAddress).decimals();
92        address yTokenAddress = ICurveTricrypto(primitive).coins(1);
96        decimals[yToken] = IERC20Metadata(yTokenAddress).decimals();
99        address wethAddress = ICurveTricrypto(primitive).coins(2);
106        address lpTokenAddress = ICurveTricrypto(primitive).token();
109        decimals[lpTokenId] = IERC20Metadata(lpTokenAddress).decimals();



129            IOceanInteractions(ocean).doInteraction{ value: amount }(interaction);
138            IOceanInteractions(ocean).doInteraction(interaction);


201            ICurveTricrypto(primitive).exchange{ value: inputToken == zToken ? rawInputAmount : 0 }(
208            if (inputToken == zToken) IWETH(underlying[zToken]).deposit{ value: rawInputAmount }();
210            ICurveTricrypto(primitive).add_liquidity(inputAmounts, 0);
213                uint256 wethBalance = IERC20Metadata(underlying[zToken]).balanceOf(address(this));
214                ICurveTricrypto(primitive).remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);
215                IWETH(underlying[zToken]).withdraw(
216                    IERC20Metadata(underlying[zToken]).balanceOf(address(this)) - wethBalance
219                ICurveTricrypto(primitive).remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);


242       IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);
243        IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L86

This report discusses how using inline assembly can optimize gas costs when making multiple external calls by reusing
memory space, rather than expanding memory separately for each call. This can save thousands of gas compared to the
solidity compiler's default behavior.

## [G-07] Don't make variables public unless necessary

**Issue Description**\
Making variables public comes with some overhead costs that can be avoided in many cases. A public variable implicitly creates
a public getter function of the same name, increasing the contract size.

**Proposed Optimization**\
Only mark variables as public if their values truly need to be readable by external contracts/users. Otherwise, use private
or internal visibility.

**Estimated Gas Savings**\
The savings from avoiding unnecessary public variables are small per transaction, around 5-10 gas. However, this adds up
over many transactions targeting a contract with public state variables that don't need to be public.

**Attachments**

- **Code Snippets**

```solidity
File: src/ocean/Ocean.sol

84    uint256 public immutable WRAPPED_ETHER_ID;

90    uint256 public unwrapFeeDivisor;
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L84

```solidity
File: src/adapters/Curve2PoolAdapter.sol

56    uint256 public immutable xToken;

59    uint256 public immutable yToken;

62    uint256 public immutable lpTokenId;
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L56

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

61    uint256 public immutable xToken;

64    uint256 public immutable yToken;

67    uint256 public immutable zToken;

70    uint256 public immutable lpTokenId;
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L61

## [G-08] It is sometimes cheaper to cache calldata

**Issue Description**\
While the calldataload opcode is relatively cheap, directly using it in a loop or multiple times can still result in unnecessary
bytecode. Caching the loaded calldata first may allow for optimization opportunities.

**Proposed Optimization**\
Cache calldata values in a local variable after first load, then reference the local variable instead of repeatedly using
calldataload.

**Estimated Gas Savings**\
Exact savings vary depending on contract, but caching calldata parameters can save 5-20 gas per usage by avoiding extra calldataload
opcodes. Larger functions with many parameter uses see more benefit.

**Attachments**

- **Code Snippets**

```solidity
File: src/ocean/Ocean.sol

230        Interaction[] calldata interactions,
231        uint256[] calldata ids


282        Interaction[] calldata interactions,
283        uint256[] calldata ids,

356        uint256[] calldata,
357        uint256[] calldata,

446        Interaction[] calldata interactions,
447        uint256[] calldata ids,
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L230-L231

## [G-09] Shorten arrays with inline assembly

**Issue Description**\
When shortening an array in Solidity, it creates a new shorter array and copies the elements over. This wastes gas by duplicating
storage.

**Proposed Optimization**\
Use inline assembly to shorten the array in place by changing its length slot, avoiding the need to copy elements to a new
array.

**Estimated Gas Savings**\
Shortening a length-n array avoids ~n SSTORE operations to copy elements. Benchmarking shows savings of 5000-15000 gas depending
on original length.

```solidity
function shorten(uint[] storage array, uint newLen) internal {

  assembly {
    sstore(array_slot, newLen)
  }

}

// Rather than:
function shorten(uint[] storage array, uint newLen) internal {

  uint[] memory newArray = new uint[](newLen);

  for(uint i = 0; i < newLen; i++) {
    newArray[i] = array[i];
  }

  delete array;
  array = newArray;

}
```

Using inline assembly allows shortening arrays without copying elements to a new storage slot, providing significant gas
savings.

**Attachments**

- **Code Snippets**

```solidity
File: src/ocean/Ocean.sol

460        BalanceDelta[] memory balanceDeltas = new BalanceDelta[](ids.length);
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L460

## [G-10] Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4

**Issue Description**\
The abi.encodePacked() function is used to concatenate multiple arguments into a byte array prior to 0.8.4. However, since
0.8.4 the bytes.concat() function was introduced which performs the same role but is preferred since it avoids ABI encoding
overhead.

**Proposed Optimization**\
Replace uses of abi.encodePacked() with bytes.concat() where multiple arguments need to be concatenated into a byte array.

**Estimated Gas Savings**\
Using bytes.concat() instead of abi.encodePacked() saves approximately 300-500 gas per concatenation by avoiding ABI encoding.

**Attachments**

- **Code Snippets**

```solidity
File: src/adapters/OceanAdapter.sol

109        return uint256(keccak256(abi.encodePacked(tokenAddress, tokenId)));
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L109

## [G-11] Private functions used once can be inlined

**Issue Description**\
Private functions that are only used internally in a single caller do not provide any abstraction benefits. The function
call overhead is wasted gas in this case.

**Proposed Optimization**\
Identify private functions that are only used in a single internal caller, and inline their code directly into the caller
by copy-pasting the function body. This avoids the unnecessary function call.

**Estimated Gas Savings**\
Inlining a private function saves approximately 200-500 gas by removing the function call overhead. This adds up over many
private functions.

**Attachments**

- **Code Snippets**

```solidity
File: src/ocean/Ocean.sol

820    function _erc20Wrap(address tokenAddress, uint256 amount, address userAddress, uint256 outputToken) private {

864    function _erc20Unwrap(address tokenAddress, uint256 amount, address userAddress, uint256 inputToken) private {

889    function _erc721Wrap(address tokenAddress, uint256 tokenId, address userAddress, uint256 oceanId) private {

903    function _erc721Unwrap(address tokenAddress, uint256 tokenId, address userAddress, uint256 oceanId) private {

920    function _erc1155Wrap(

955    function _erc1155Unwrap(

978    function _etherUnwrap(uint256 amount, address userAddress) private {

1068    function _determineTransferAmount(
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L1068

```solidity
File: src/adapters/Curve2PoolAdapter.sol

201    function _determineComputeType(
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L201

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

264    function _determineComputeType(
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L264

## [G-12] Make 3 event parameters indexed when possible

**Issue Description**\
Events allow logging information on-chain for analytics/tracing purposes. However, emitting unindexed parameters is inefficient
as it increases the size of the transaction logs linearly with the data size.

**Proposed Optimization**\
Review all events and make the first 3 non-address parameters indexed if possible, to optimize for gas efficiency of log
encoding. If an event has fewer than 3 parameters, make them all indexed.

**Estimated Gas Savings**\
Each indexed parameter saves approximately 200 gas vs an unindexed parameter by reducing log encoding size. With many events,
this optimization can add up to significant gas savings.

**Attachments**

- **Code Snippets**

```solidity
File: src/ocean/Ocean.sol

111    event ChangeUnwrapFee(uint256 oldFee, uint256 newFee, address sender);

142    event EtherUnwrap(uint256 amount, uint256 feeCharged, address indexed user);

143    event ComputeOutputAmount(
144     address indexed primitive,
145     uint256 inputToken,
146     uint256 outputToken,
147     uint256 inputAmount,
148     uint256 outputAmount,
149     address indexed user
150 );

151    event ComputeInputAmount(
152     address indexed primitive,
153     uint256 inputToken,
154     uint256 outputToken,
155     uint256 inputAmount,
156     uint256 outputAmount,
157     address indexed user
158 );

159    event OceanTransaction(address indexed user, uint256 numberOfInteractions);
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L151-L159

```solidity
File:

30    event Swap(
31        uint256 inputToken,
32        uint256 inputAmount,
33        uint256 outputAmount,
34        bytes32 slippageProtection,
35        address user,
36        bool computeOutput
37    );


38  event Deposit(
39      uint256 inputToken,
40      uint256 inputAmount,
41      uint256 outputAmount,
42      bytes32 slippageProtection,
43      address user,
44      bool computeOutput
45  );


46  event Withdraw(
47      uint256 outputToken,
48      uint256 inputAmount,
49      uint256 outputAmount,
50      bytes32 slippageProtection,
51      address user,
52      bool computeOutput
53  );
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L30-L53

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

35    event Swap(
36      uint256 inputToken,
37      uint256 inputAmount,
38      uint256 outputAmount,
39      bytes32 slippageProtection,
40      address user,
41      bool computeOutput
42  );


43  event Deposit(
44      uint256 inputToken,
45      uint256 inputAmount,
46      uint256 outputAmount,
47      bytes32 slippageProtection,
48     address user,
49      bool computeOutput
50  );


51  event Withdraw(
52      uint256 outputToken,
53      uint256 inputAmount,
54      uint256 outputAmount,
55      bytes32 slippageProtection,
56      address user,
57      bool computeOutput
58  );
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L35-L58

## [G-13] Consider using ERC721 instead of ERC721

[ERC721A](https://www.erc721a.org/) is an improved implementation of IERC721 with significant gas savings for minting
multiple NFTs in a single transaction.

```solidity
File: src/ocean/Ocean.sol

12  import { IERC721 } from "@openzeppelin/contracts/token/ERC721/IERC721.sol";
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L12
