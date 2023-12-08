## \[G-01\] Use assembly to validate `msg.sender`

We can use assembly to efficiently validate msg.sender for the didPay and uniswapV3SwapCallback functions with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs.

```
File: src/adapters/OceanAdapter.sol

39: require(msg.sender == ocean);
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L39
## \[G-02\] uint8 is not always cheaper than uint256

The EVM only operates on 32 bytes/ 256 bits at a time. This means that if you use uint8, EVM has to first convert it uint256 to work on it and the conversion costs extra gas! You may wonder, What were the devs thinking? Why did they create smaller variables then? The answer lies in packing. In solidity, you can pack multiple small variables into one slot, but if you are defining a lone variable and can’t pack it, it’s optimal to use a uint256 rather than uint8.

```
16:  uint8 constant NORMALIZED_DECIMALS = 18;
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L16

## \[G‑03\] Using `bool`s for storage incurs overhead

```
// Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.
```

Use `uint256(1)` and `uint256(2)` for true/false to avoid a Gwarmaccess (**<ins>100 gas</ins>**) for the extra SLOAD, and to avoid Gsset (**20000 gas**) when changing from `false` to `true`, after having been `true` in the past.

```
File: src/adapters/Curve2PoolAdapter.sol

36: bool computeOutput

44: bool computeOutput

52: bool computeOutput
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L36

## \[G-04\] Use hardcode address instead `address(this)`

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded `address`. Foundry’s script.sol and solmate’s `LibRlp.sol` contracts can help achieve this.

References: <ins>https://book.getfoundry.sh/reference/forge-std/compute-create-address</ins>

<ins>https://twitter.com/transmissions11/status/1518507047943245824</ins>

```
253: return IERC20Metadata(tokenAddress).balanceOf(address(this));
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L253

### Recommended Mitigation Steps

Use hardcoded `address`.

## \[G-05\] Use constants instead of type(uintx).max

it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.

Here's an example of how you can use a constant instead of type(uintX).max:

```
contract MyContract {
    uint120 constant MAX_VALUE = 2**120 - 1;
    
    function doSomething(uint120 value) public {
        require(value <= MAX_VALUE, "Value exceeds maximum");
        
        // Do something
    }
}
```

In the above example, we have a contract with a constant MAX\_VALUE that represents the maximum value of a uint120. When the doSomething function is called with a value parameter, it checks whether the value is less than or equal to MAX\_VALUE using the <= operator.

By using a constant instead of type(uint120).max, we can make our code more efficient and reduce the gas cost of our contract.

It's important to note that using constants can make your code more readable and maintainable, since the value is defined in one place and can be easily updated if necessary. However, constants should be used with caution and only when their value is known at compile-time.

```
function _approveToken(address tokenAddress) private {
        IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);
        IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);
    }
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L189

```
uint256 constant GET_BALANCE_DELTA = type(uint256).max;
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L102

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L170

## \[G-06\] Use `ERC721A` instead `ERC721`

### Mitigation

`ERC721A` is an improvement standard for `ERC721` tokens. It was proposed by the Azuki team and used for developing their NFT collection. Compared with `ERC721`, `ERC721A` is a more gas-efficient standard to mint a lot of of NFTs simultaneously. It allows developers to mint multiple NFTs at the same gas price. This has been a great improvement due to Ethereum’s sky-rocketing gas fee. Reference: <ins>https://nextrope.com/erc721-vs-erc721a-2/</ins>.

```
import { IERC721 } from "@openzeppelin/contracts/token/ERC721/IERC721.sol";
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L12

## \[G-07\] Return values from external calls can be cached to avoid unnecessary call

External calls are expensive as they use the `STATICCALL`/`CALL` opcode (~100 gas). If you are calling the same external function more than once you should cache the return value to avoid an unnecessary `STATICCALL`/`CALL`.

```
244: return _doMultipleInteractions(interactions, ids, msg.sender);
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L244

```
267: return _doInteraction(interaction, userAddress);
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L267

## \[G-08\] Make 3 event parameters indexed when possible

It’s the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

```
event EtherWrap(uint256 amount, address indexed user);
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L141

```
event EtherUnwrap(uint256 amount, uint256 feeCharged, address indexed user);
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L142

```
event ComputeOutputAmount(
        address indexed primitive,
        uint256 inputToken,
        uint256 outputToken,
        uint256 inputAmount,
        uint256 outputAmount,
        address indexed user
    );
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L143C5-L150C7

```
event ComputeInputAmount(
        address indexed primitive,
        uint256 inputToken,
        uint256 outputToken,
        uint256 inputAmount,
        uint256 outputAmount,
        address indexed user
    );
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L151C3-L158C7

```
event OceanTransaction(address indexed user, uint256 numberOfInteractions);
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L159

```
event ForwardedOceanTransaction(address indexed forwarder, address indexed user, uint256 numberOfInteractions);
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L160