**Note:** I've removed all finding instances appearing the bot and 4naly3er reports, and have only kept the ones that they missed

## Summary

### Gas Optimizations

| |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [[G&#x2011;01](#g01-enums-cost-more-gas-to-use-than-uint256s)] | `enum`s cost more gas to use than `uint256`s | 2 |  - |
| [[G&#x2011;02](#g02-internalprivate-functions-only-called-once-can-be-inlined-to-save-gas)] | `internal`/`private` functions only called once can be inlined to save gas | 10 |  200 |
| [[G&#x2011;03](#g03-assembly-check-msgsender-using-xor-and-the-scratch-space)] | Assembly: Check `msg.sender` using `xor` and the scratch space | 1 |  - |
| [[G&#x2011;04](#g04-assembly-use-scratch-space-for-building-calldata)] | Assembly: Use scratch space for building calldata | 5 |  1100 |
| [[G&#x2011;05](#g05-assembly-use-scratch-space-when-building-emitted-events-with-two-data-arguments)] | Assembly: Use scratch space when building emitted events with two data arguments | 2 |  30 |
| [[G&#x2011;06](#g06-emitting-constants-wastes-gas)] | Emitting constants wastes gas | 8 |  64 |
| [[G&#x2011;07](#g07-reduce-deployment-costs-by-tweaking-contracts-metadata)] | Reduce deployment costs by tweaking contracts' metadata | 3 |  - |
| [[G&#x2011;08](#g08-stack-variable-is-only-used-once)] | Stack variable is only used once | 18 |  54 |
| [[G&#x2011;09](#g09-splitting-require-statements-that-use--saves-gas)] | Splitting `require()` statements that use `&&` saves gas | 1 |  3 |
| [[G&#x2011;10](#g10-using-private-rather-than-public-saves-gas)] | Using `private` rather than `public`, saves gas | 12 |  - |

Total: 62 instances over 10 issues with **1451 gas** saved

Gas totals are estimates based on data from the Ethereum Yellowpaper. The estimates use the lower bounds of ranges and count two iterations of each `for`-loop. All values above are runtime, not deployment, values; deployment values are listed in the individual issue descriptions. The table above as well as its gas numbers do not include any of the excluded findings.





## Gas Optimizations


### [G&#x2011;01] `enum`s cost more gas to use than `uint256`s
Enums are equivalent in size to `uint8`s, which the overhead of having to clear the upper bits.

*There are 2 instances of this issue:*

```solidity
File: src/adapters/Curve2PoolAdapter.sol

10   enum ComputeType {
11       Deposit,
12       Swap,
13       Withdraw
14:  }

```
*GitHub*: [10](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L10-L14)

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

15   enum ComputeType {
16       Deposit,
17       Swap,
18       Withdraw
19:  }

```
*GitHub*: [15](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L15-L19)



### [G&#x2011;02] `internal`/`private` functions only called once can be inlined to save gas
Not inlining costs **20 to 40 gas** because of two extra `JUMP` instructions and additional stack operations needed for function calls. The inliner can do it only for 'simple' cases:
> Now to get back to the point why we require the routine to be simple: As soon as you do more complicated things like for example branching, calling external contracts, the Common Subexpression Eliminator cannot re-construct the code anymore or does not do full symbolic evaluation of the expressions. 

https://soliditylang.org/blog/2021/03/02/saving-gas-with-simple-inliner/

Therefore, the instances below contain branching or use op-codes with side-effects

*There are 10 instances of this issue:*

```solidity
File: src/adapters/Curve2PoolAdapter.sol

201      function _determineComputeType(
202          uint256 inputToken,
203          uint256 outputToken
204      )
205          private
206          view
207          returns (ComputeType computeType)
208:     {

```
*GitHub*: [201](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L201-L208)

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

264      function _determineComputeType(
265          uint256 inputToken,
266          uint256 outputToken
267      )
268          private
269          view
270          returns (ComputeType computeType)
271:     {

```
*GitHub*: [264](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L264-L271)

```solidity
File: src/ocean/Ocean.sol

820:     function _erc20Wrap(address tokenAddress, uint256 amount, address userAddress, uint256 outputToken) private {

864:     function _erc20Unwrap(address tokenAddress, uint256 amount, address userAddress, uint256 inputToken) private {

889:     function _erc721Wrap(address tokenAddress, uint256 tokenId, address userAddress, uint256 oceanId) private {

903:     function _erc721Unwrap(address tokenAddress, uint256 tokenId, address userAddress, uint256 oceanId) private {

920      function _erc1155Wrap(
921          address tokenAddress,
922          uint256 tokenId,
923          uint256 amount,
924          address userAddress,
925          uint256 oceanId
926      )
927          private
928:     {

955      function _erc1155Unwrap(
956          address tokenAddress,
957          uint256 tokenId,
958          uint256 amount,
959          address userAddress,
960          uint256 oceanId
961      )
962          private
963:     {

978:     function _etherUnwrap(uint256 amount, address userAddress) private {

1068     function _determineTransferAmount(
1069         uint256 amount,
1070         uint8 decimals
1071     )
1072         private
1073         pure
1074         returns (uint256 transferAmount, uint256 dust)
1075:    {

```
*GitHub*: [820](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L820-L820), [864](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L864-L864), [889](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L889-L889), [903](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L903-L903), [920](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L920-L928), [955](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L955-L963), [978](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L978-L978), [1068](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L1068-L1075)



### [G&#x2011;03] Assembly: Check `msg.sender` using `xor` and the scratch space
See [this](https://code4rena.com/reports/2023-05-juicebox#g-06-use-assembly-to-validate-msgsender) prior finding for details on the conversion

*There is one instance of this issue:*

```solidity
File: src/adapters/OceanAdapter.sol

39:          require(msg.sender == ocean);

```
*GitHub*: [39](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L39-L39)



### [G&#x2011;04] Assembly: Use scratch space for building calldata
If an external call's calldata can fit into two or fewer words, use the scratch space to build the calldata, rather than allowing Solidity to do a memory expansion.

*There are 5 instances of this issue:*

```solidity
File: src/adapters/Curve2PoolAdapter.sol

190:         IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);

191:         IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);

```
*GitHub*: [190](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L190-L190), [191](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L191-L191)

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

242:         IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);

243:         IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);

253:             return IERC20Metadata(tokenAddress).balanceOf(address(this));

```
*GitHub*: [242](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L242-L242), [243](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L243-L243), [253](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L253-L253)



### [G&#x2011;05] Assembly: Use scratch space when building emitted events with two data arguments
Using the [scratch space](https://gist.github.com/IllIllI000/87c4f03139fa03780fa548b8e4b02b5b) for more than one, but at most two words worth of data (non-indexed arguments) will save gas over needing Solidity's abi memory expansion used for emitting normally.

*There are 2 instances of this issue:*

```solidity
File: src/ocean/Ocean.sol

933:         emit Erc1155Wrap(tokenAddress, tokenId, amount, userAddress, oceanId);

983:         emit EtherUnwrap(transferAmount, feeCharged, userAddress);

```
*GitHub*: [933](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L933-L933), [983](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L983-L983)



### [G&#x2011;06] Emitting constants wastes gas
Every event parameter costs `Glogdata` (**8 gas**) per byte. You can avoid this extra cost, in cases where you're emitting a constant, by creating a second version of the event, which doesn't have the parameter (and have users look to the contract's variables for its value instead), and using the new event in the cases shown below.

*There are 8 instances of this issue:*

```solidity
File: src/adapters/Curve2PoolAdapter.sol

178:             emit Swap(inputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);

180:             emit Deposit(inputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);

182:             emit Withdraw(outputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);

```
*GitHub*: [178](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L178-L178), [180](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L180-L180), [182](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L182-L182)

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

230:             emit Swap(inputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);

232:             emit Deposit(inputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);

234:             emit Withdraw(outputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);

```
*GitHub*: [230](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L230-L230), [232](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L232-L232), [234](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L234-L234)

```solidity
File: src/ocean/Ocean.sol

216:         emit OceanTransaction(msg.sender, 1);

266:         emit ForwardedOceanTransaction(msg.sender, userAddress, 1);

```
*GitHub*: [216](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L216-L216), [266](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L266-L266)



### [G&#x2011;07] Reduce deployment costs by tweaking contracts' metadata
See [this](https://www.rareskills.io/post/solidity-metadata) link, at its bottom, for full details

*There are 3 instances of this issue:*

```solidity
File: src/adapters/Curve2PoolAdapter.sol

20   contract Curve2PoolAdapter is OceanAdapter {
21       /////////////////////////////////////////////////////////////////////
22       //                             Errors                              //
23:      /////////////////////////////////////////////////////////////////////

```
*GitHub*: [20](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L20-L23)

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

25   contract CurveTricryptoAdapter is OceanAdapter {
26       /////////////////////////////////////////////////////////////////////
27       //                             Errors                              //
28:      /////////////////////////////////////////////////////////////////////

```
*GitHub*: [25](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L25-L28)

```solidity
File: src/ocean/Ocean.sol

79:  contract Ocean is IOceanInteractions, IOceanFeeChange, OceanERC1155, IERC721Receiver, IERC1155Receiver {

```
*GitHub*: [79](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L79-L79)



### [G&#x2011;08] Stack variable is only used once
If the variable is only accessed once, it's cheaper to use the assigned value directly that one time, and save the **3 gas** the extra stack assignment would spend

*There are 18 instances of this issue:*

```solidity
File: src/adapters/Curve2PoolAdapter.sol

103:         address tokenAddress = underlying[tokenId];

105          Interaction memory interaction = Interaction({
106              interactionTypeAndAddress: _fetchInteractionId(tokenAddress, uint256(InteractionType.WrapErc20)),
107              inputToken: 0,
108              outputToken: 0,
109              specifiedAmount: amount,
110              metadata: bytes32(0)
111:         });

122:         address tokenAddress = underlying[tokenId];

124          Interaction memory interaction = Interaction({
125              interactionTypeAndAddress: _fetchInteractionId(tokenAddress, uint256(InteractionType.UnwrapErc20)),
126              inputToken: 0,
127              outputToken: 0,
128              specifiedAmount: amount,
129              metadata: bytes32(0)
130:         });

```
*GitHub*: [103](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L103-L103), [105](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L105-L111), [122](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L122-L122), [124](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L124-L130)

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

192:         uint256 _balanceBefore = _getBalance(underlying[outputToken]);

199:             bool useEth = inputToken == zToken || outputToken == zToken;

213:                 uint256 wethBalance = IERC20Metadata(underlying[zToken]).balanceOf(address(this));

223:         uint256 rawOutputAmount = _getBalance(underlying[outputToken]) - _balanceBefore;

```
*GitHub*: [192](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L192-L192), [199](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L199-L199), [213](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L213-L213), [223](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L223-L223)

```solidity
File: src/adapters/OceanAdapter.sol

70:          uint256 unwrapFee = inputAmount / IOceanInteractions(ocean).unwrapFeeDivisor();

71:          uint256 unwrappedAmount = inputAmount - unwrapFee;

152:             uint256 shift = 10 ** (uint256(decimalsTo - decimalsFrom));

156:             uint256 shift = 10 ** (uint256(decimalsFrom - decimalsTo));

```
*GitHub*: [70](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L70-L70), [71](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L71-L71), [152](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L152-L152), [156](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L156-L156)

```solidity
File: src/ocean/Ocean.sol

405:             uint256 specifiedToken = _getSpecifiedToken(interactionType, externalContract, interaction);

867:             uint256 amountRemaining = amount - feeCharged;

869              (uint256 transferAmount, uint256 truncated) =
870:                 _convertDecimals(NORMALIZED_DECIMALS, decimals, amountRemaining);

966:         uint256 amountRemaining = amount - feeCharged;

1096             (uint256 normalizedTransferAmount, uint256 normalizedTruncatedAmount) =
1097:                _convertDecimals(decimals, NORMALIZED_DECIMALS, transferAmount);

1138:            uint256 shift = 10 ** (uint256(decimalsTo - decimalsFrom));

```
*GitHub*: [405](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L405-L405), [867](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L867-L867), [869](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L869-L870), [966](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L966-L966), [1096](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L1096-L1097), [1138](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L1138-L1138)



### [G&#x2011;09] Splitting `require()` statements that use `&&` saves gas
See [this issue](https://github.com/code-423n4/2022-01-xdefi-findings/issues/128) which describes the fact that there is a larger deployment gas cost, but with enough runtime calls, the change ends up being cheaper by **3 gas**

*There is one instance of this issue:*

```solidity
File: src/ocean/Ocean.sol

667:             assert(interactionType == InteractionType.UnwrapEther && specifiedToken == WRAPPED_ETHER_ID);

```
*GitHub*: [667](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L667-L667)



### [G&#x2011;10] Using `private` rather than `public`, saves gas
For constants, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*There are 12 instances of this issue:*

```solidity
File: src/adapters/Curve2PoolAdapter.sol

56:      uint256 public immutable xToken;

59:      uint256 public immutable yToken;

62:      uint256 public immutable lpTokenId;

```
*GitHub*: [56](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L56-L56), [59](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L59-L59), [62](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L62-L62)

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

61:      uint256 public immutable xToken;

64:      uint256 public immutable yToken;

67:      uint256 public immutable zToken;

70:      uint256 public immutable lpTokenId;

```
*GitHub*: [61](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L61-L61), [64](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L64-L64), [67](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L67-L67), [70](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L70-L70)

```solidity
File: src/adapters/OceanAdapter.sol

19:      address public immutable ocean;

22:      address public immutable primitive;

25:      mapping(uint256 => address) public underlying;

```
*GitHub*: [19](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L19-L19), [22](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L22-L22), [25](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L25-L25)

```solidity
File: src/ocean/Ocean.sol

84:      uint256 public immutable WRAPPED_ETHER_ID;

90:      uint256 public unwrapFeeDivisor;

```
*GitHub*: [84](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L84-L84), [90](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L90-L90)
