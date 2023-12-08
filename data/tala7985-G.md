## \[G‑1\] Newer versions of solidity are more gas efficient

The solidity language continues to pursue more efficient gas optimization schemes. Adopting a <ins>newer version of solidity</ins> can be more gas efficient.

```
pragma solidity 0.8.20;
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L6

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L4

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L4

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L4

### \[G‑2\] Consider using `ERC721A` instead of `ERC721`

<ins>ERC721A</ins> is an improved implementation of IERC721 with significant gas savings for minting multiple NFTs in a single transaction.

```
import { IERC721 } from "@openzeppelin/contracts/token/ERC721/IERC721.sol";
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L12

## \[G-3\] Make 3 event parameters indexed when possible

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

## \[G‑4\] Using assembly to check for zero can save gas

Using assembly to check for zero can save gas by allowing more direct access to the evm and reducing some of the overhead associated with high-level operations in solidity.

```
if (msg.value != 0) {
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L391

```
if (msg.value != 0) {
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L474

## \[G-5\] += costs more gas than = + for state variables

```
feeCharged += truncated;
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L871

## \[G‑6\] Using `constant`s instead of `==enum==` can save gas

`==Enum==` is expensive and it is <ins>more efficient to use constants</ins> instead. An illustrative example of this approach can be found in the <ins>ReentrancyGuard.sol</ins>.

```
enum ComputeType {
    Deposit,
    Swap,
    Withdraw
}
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L10C1-L14C2

```
enum ComputeType {
    Deposit,
    Swap,
    Withdraw
}
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L15C1-L19C2

## \[G‑7\] internal functions not called by the contract should be removed to save deployment gas

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword.

```
function _fetchInteractionId(address token, uint256 interactionType) internal pure returns (bytes32) {
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L99

```
function _calculateOceanId(address tokenAddress, uint256 tokenId) internal pure returns (uint256) {
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L108

## \[G-8\]  Public functions not called in the contract should be marked External

Declaring functions as public costs more gas. Replace them with external wherever possible.

```
function onERC1155Received(address, address, uint256, uint256, bytes memory) public pure returns (bytes4) {
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L117C14-L117C31

## \[G-9\] Parameter not used in the function

### [Description](https://github.com/code-423n4/2022-08-frax-findings/blob/main/data/ajtra-G.md#description-14)

Remove the parameter \_initData in VariableInterestRate.getNewRate.\_initData because of it's not used in the function.

```
function getTokenSupply(uint256 tokenId) external view override returns (uint256) {
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L124

## \[G-10\] Use constants instead of type(uintx).max

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

## \[G‑11\] Using `bool`s for storage incurs overhead

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

## \[G-12\] Use assembly to validate `msg.sender`

We can use assembly to efficiently validate msg.sender for the didPay and uniswapV3SwapCallback functions with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs.

```
File: src/adapters/OceanAdapter.sol

41: require(msg.sender == ocean);
```