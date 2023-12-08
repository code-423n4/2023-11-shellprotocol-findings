# Gas Optimization

# Summary

| Number | Gas Optimization                                                                                | Context |
| :----: | :---------------------------------------------------------------------------------------------- | :-----: |
| [G-01] | Add unchecked {} for subtractions where the operands can't underflow because of previous checks |    2    |
| [G-02] | Use constants instead of type(uintx).max                                                        |    5    |
| [G-03] | Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4           |    1    |
| [G-04] | Make 3 event parameters indexed when possible                                                   |   10    |
| [G-05] | Private functions used once can be inlined                                                      |   10    |

## [G-01] Add unchecked {} for subtractions where the operands can't underflow because of previous checks

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

## [G-02] Use constants instead of type(uintx).max

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

242        IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);

243        IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L242-L243

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

## [G-03] Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4

```solidity
File: src/adapters/OceanAdapter.sol

109        return uint256(keccak256(abi.encodePacked(tokenAddress, tokenId)));
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L109

## [G-04] Make 3 event parameters indexed when possible

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
File: src/adapters/Curve2PoolAdapter.sol

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

## [G-05] Private functions used once can be inlined

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