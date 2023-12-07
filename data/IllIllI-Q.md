**Note:** I've removed all finding instances appearing the bot and 4naly3er reports, and have only kept the ones that they missed

## Summary

### Low Risk Issues

| |Issue|Instances|
|-|:-|:-:|
| [[L&#x2011;01](#l01-receivepayable-fallback-function-does-not-authorize-requests)] | `receive()`/`payable fallback()` function does not authorize requests | 1 | 

Total: 1 instances over 1 issues

### Non-critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [[N&#x2011;01](#n01-if-statement-can-be-converted-to-a-ternary)] | `if`-statement can be converted to a ternary | 2 | 
| [[N&#x2011;02](#n02-public-functions-not-called-by-the-contract-should-be-declared-external-instead)] | `public` functions not called by the contract should be declared `external` instead | 1 | 
| [[N&#x2011;03](#n03-consider-defining-system-wide-constants-in-a-single-file)] | Consider defining system-wide constants in a single file | 1 | 
| [[N&#x2011;04](#n04-consider-making-contracts-upgradeable)] | Consider making contracts `Upgradeable` | 3 | 
| [[N&#x2011;05](#n05-contract-should-expose-an-interface)] | Contract should expose an `interface` | 1 | 
| [[N&#x2011;06](#n06-contract-uses-both-requirerevert-as-well-as-custom-errors)] | Contract uses both `require()`/`revert()` as well as custom errors | 1 | 
| [[N&#x2011;07](#n07-contractslibrariesinterfaces-should-each-be-defined-in-separate-files)] | Contracts/libraries/interfaces should each be defined in separate files | 1 | 
| [[N&#x2011;08](#n08-duplicated-requirerevert-checks-should-be-refactored-to-a-modifier-or-function)] | Duplicated `require()`/`revert()` checks should be refactored to a modifier or function | 1 | 
| [[N&#x2011;09](#n09-interfaces-should-be-defined-in-separate-files-from-their-usage)] | Interfaces should be defined in separate files from their usage | 1 | 
| [[N&#x2011;10](#n10-interfaces-should-use-floating-version-pragmas)] | Interfaces should use floating version pragmas | 1 | 
| [[N&#x2011;11](#n11-long-functions-should-be-refactored-into-multiple-smaller-functions)] | Long functions should be refactored into multiple, smaller, functions | 4 | 
| [[N&#x2011;12](#n12-named-imports-of-parent-contracts-are-missing)] | Named imports of parent contracts are missing | 3 | 
| [[N&#x2011;13](#n13-natspec-error-declarations-should-have-descriptions)] | NatSpec: Error declarations should have descriptions | 2 | 
| [[N&#x2011;14](#n14-natspec-event-declarations-should-have-descriptions)] | NatSpec: Event declarations should have descriptions | 2 | 
| [[N&#x2011;15](#n15-natspec-function-param-tag-is-missing)] | NatSpec: Function `@param` tag is missing | 48 | 
| [[N&#x2011;16](#n16-natspec-function-return-tag-is-missing)] | NatSpec: Function `@return` tag is missing | 41 | 
| [[N&#x2011;17](#n17-natspec-function-declarations-should-have-notice-tags)] | NatSpec: Function declarations should have `@notice` tags | 4 | 
| [[N&#x2011;18](#n18-natspec-non-public-state-variable-declarations-should-use-dev-tags)] | NatSpec: Non-public state variable declarations should use `@dev` tags | 6 | 
| [[N&#x2011;19](#n19-style-guide-extraneous-whitespace)] | Style guide: Extraneous whitespace | 16 | 
| [[N&#x2011;20](#n20-style-guide-non-externalpublic-variable-names-should-begin-with-an-underscore)] | Style guide: Non-`external`/`public` variable names should begin with an underscore | 4 | 
| [[N&#x2011;21](#n21-unused-function-parameter)] | Unused function parameter | 6 | 
| [[N&#x2011;22](#n22-use-of-override-is-unnecessary)] | Use of `override` is unnecessary | 18 | 

Total: 167 instances over 22 issues




## Low Risk Issues


### [L&#x2011;01] `receive()`/`payable fallback()` function does not authorize requests
Having no access control on the function (e.g. `require(msg.sender == address(weth))`) means that someone may send Ether to the contract, and have no way to get anything back out, which is a loss of funds. If the concern is having to spend a small amount of gas to check the sender against an immutable address, the code should at least have a function to rescue mistakenly-sent Ether.

*There is one instance of this issue:*

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

291:      fallback() external payable { }

```
*GitHub*: [291](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L291)


## Non-critical Issues


### [N&#x2011;01] `if`-statement can be converted to a ternary
The code can be made more compact while also increasing readability by converting the following `if`-statements to ternaries (e.g. `foo += (x > y) ? a : b`)

*There are 2 instances of this issue:*

```solidity
File: src/ocean/Ocean.sol

310          if (_ERC721InteractionStatus == INTERACTION) {
311              return IERC721Receiver.onERC721Received.selector;
312          } else {
313              return 0;
314:         }

515                  if (interaction.specifiedAmount == GET_BALANCE_DELTA) {
516                      specifiedAmount = balanceDeltas.getBalanceDelta(interactionType, specifiedToken);
517                  } else {
518                      specifiedAmount = interaction.specifiedAmount;
519:                 }

```
*GitHub*: [310](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L310-L314), [515](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L515-L519)



### [N&#x2011;02] `public` functions not called by the contract should be declared `external` instead
Contracts [are allowed](https://docs.soliditylang.org/en/latest/contracts.html#function-overriding) to override their parents' functions and change the visibility from `external` to `public`.

*There is one instance of this issue:*

```solidity
File: src/ocean/Ocean.sol

305:      function supportsInterface(bytes4 interfaceId) public view virtual override(OceanERC1155, IERC165) returns (bool) {

```
*GitHub*: [305](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L305)



### [N&#x2011;03] Consider defining system-wide constants in a single file


*There is one instance of this issue:*

```solidity
File: src/adapters/OceanAdapter.sol

16:      uint8 constant NORMALIZED_DECIMALS = 18;

```
*GitHub*: [16](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L16-L16)



### [N&#x2011;04] Consider making contracts `Upgradeable`
This allows for bugs to be fixed in production, at the expense of _significantly_ increasing centralization.

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



### [N&#x2011;05] Contract should expose an `interface`
The `contract`s should expose an `interface` so that other projects can more easily integrate with it, without having to develop their own non-standard variants.

*There is one instance of this issue:*

```solidity
File: src/ocean/Ocean.sol

/// @audit  WRAPPED_ETHER_ID(), unwrapFeeDivisor()
79:  contract Ocean is IOceanInteractions, IOceanFeeChange, OceanERC1155, IERC721Receiver, IERC1155Receiver {

```
*GitHub*: [79](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L79-L79)



### [N&#x2011;06] Contract uses both `require()`/`revert()` as well as custom errors
Consider using just one method in a single file

*There is one instance of this issue:*

```solidity
File: src/ocean/Ocean.sol

79:  contract Ocean is IOceanInteractions, IOceanFeeChange, OceanERC1155, IERC721Receiver, IERC1155Receiver {

```
*GitHub*: [79](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L79-L79)



### [N&#x2011;07] Contracts/libraries/interfaces should each be defined in separate files
This helps to make tracking changes across commits easier, among other reasons. They can be combined into a single file later by importing multiple contracts, and using that file as the common import. The instances below are the second+ contract/library/interface within each file

*There is one instance of this issue:*

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

25   contract CurveTricryptoAdapter is OceanAdapter {
26       /////////////////////////////////////////////////////////////////////
27       //                             Errors                              //
28:      /////////////////////////////////////////////////////////////////////

```
*GitHub*: [25](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L25-L28)



### [N&#x2011;08] Duplicated `require()`/`revert()` checks should be refactored to a modifier or function
The compiler will inline the function, which will avoid `JUMP` instructions usually associated with functions

*There is one instance of this issue:*

```solidity
File: src/ocean/Ocean.sol

648:              if (specifiedAmount != 1) revert INVALID_ERC721_AMOUNT();

```
*GitHub*: [648](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L648)



### [N&#x2011;09] Interfaces should be defined in separate files from their usage
The interfaces below should be defined in separate files, so that it's easier for future projects to import them, and to avoid duplication later on if they need to be used elsewhere in the project

*There is one instance of this issue:*

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

10:  interface IWETH {

```
*GitHub*: [10](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L10-L10)



### [N&#x2011;10] Interfaces should use floating version pragmas
If the file contains non-interfaces as well, the interfaces should be moved to a separate file.

*There is one instance of this issue:*

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

/// @audit IWETH
4:   pragma solidity 0.8.20;

```
*GitHub*: [4](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L4-L4)



### [N&#x2011;11] Long functions should be refactored into multiple, smaller, functions


*There are 4 instances of this issue:*

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

/// @audit 59 lines (48 in the body)
178       function primitiveOutputAmount(
179           uint256 inputToken,
180           uint256 outputToken,
181           uint256 inputAmount,
182           bytes32 minimumOutputAmount
183       )
184           internal
185           override
186:          returns (uint256 outputAmount)

```
*GitHub*: [178](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L178-L186)

```solidity
File: src/ocean/Ocean.sol

/// @audit 51 lines (43 in the body)
380       function _doInteraction(
381           Interaction calldata interaction,
382           address userAddress
383       )
384           internal
385:          returns (uint256 inputToken, uint256 inputAmount, uint256 outputToken, uint256 outputAmount)

/// @audit 129 lines (115 in the body)
445       function _doMultipleInteractions(
446           Interaction[] calldata interactions,
447           uint256[] calldata ids,
448           address userAddress
449       )
450           internal
451           returns (
452               uint256[] memory burnIds,
453               uint256[] memory burnAmounts,
454               uint256[] memory mintIds,
455:              uint256[] memory mintAmounts

/// @audit 78 lines (66 in the body)
597       function _executeInteraction(
598           Interaction memory interaction,
599           InteractionType interactionType,
600           address externalContract,
601           uint256 specifiedToken,
602           uint256 specifiedAmount,
603           address userAddress
604       )
605           internal
606:          returns (uint256 inputToken, uint256 inputAmount, uint256 outputToken, uint256 outputAmount)

```
*GitHub*: [380](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L380-L385), [445](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L445-L455), [597](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L597-L606)



### [N&#x2011;12] Named imports of parent contracts are missing


*There are 3 instances of this issue:*

```solidity
File: src/adapters/Curve2PoolAdapter.sol

/// @audit OceanAdapter
20:  contract Curve2PoolAdapter is OceanAdapter {

```
*GitHub*: [20](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L20-L20)

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

/// @audit OceanAdapter
25:  contract CurveTricryptoAdapter is OceanAdapter {

```
*GitHub*: [25](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L25-L25)

```solidity
File: src/adapters/OceanAdapter.sol

/// @audit IOceanPrimitive
14:  abstract contract OceanAdapter is IOceanPrimitive {

```
*GitHub*: [14](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L14-L14)



### [N&#x2011;13] NatSpec: Error declarations should have descriptions


*There are 2 instances of this issue:*

```solidity
File: src/adapters/Curve2PoolAdapter.sol

24:      error INVALID_COMPUTE_TYPE();

```
*GitHub*: [24](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L24-L24)

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

29:      error INVALID_COMPUTE_TYPE();

```
*GitHub*: [29](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L29-L29)



### [N&#x2011;14] NatSpec: Event declarations should have descriptions


*There are 2 instances of this issue:*

```solidity
File: src/adapters/Curve2PoolAdapter.sol

30       event Swap(
31           uint256 inputToken,
32           uint256 inputAmount,
33           uint256 outputAmount,
34           bytes32 slippageProtection,
35           address user,
36           bool computeOutput
37:      );

```
*GitHub*: [30](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L30-L37)

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

35       event Swap(
36           uint256 inputToken,
37           uint256 inputAmount,
38           uint256 outputAmount,
39           bytes32 slippageProtection,
40           address user,
41           bool computeOutput
42:      );

```
*GitHub*: [35](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L35-L42)



### [N&#x2011;15] NatSpec: Function `@param` tag is missing


*There are 48 instances of this issue:*

```solidity
File: src/adapters/Curve2PoolAdapter.sol

/// @audit Missing '@param ocean_'
/// @audit Missing '@param primitive_'
74       /**
75        * @notice only initializing the immutables, mappings & approves tokens
76        */
77:      constructor(address ocean_, address primitive_) OceanAdapter(ocean_, primitive_) {

/// @audit Missing '@param tokenAddress'
186      /**
187       * @dev Approves token to be spent by the Ocean and the Curve pool
188       */
189:     function _approveToken(address tokenAddress) private {

/// @audit Missing '@param inputToken'
194      /**
195       * @dev Uses the inputToken and outputToken to determine the ComputeType
196       *  (input: xToken, output: yToken) | (input: yToken, output: xToken) => SWAP
197       *  base := xToken | yToken
198       *  (input: base, output: lpToken) => DEPOSIT
199       *  (input: lpToken, output: base) => WITHDRAW
200       */
201      function _determineComputeType(
202:         uint256 inputToken,

/// @audit Missing '@param outputToken'
194      /**
195       * @dev Uses the inputToken and outputToken to determine the ComputeType
196       *  (input: xToken, output: yToken) | (input: yToken, output: xToken) => SWAP
197       *  base := xToken | yToken
198       *  (input: base, output: lpToken) => DEPOSIT
199       *  (input: lpToken, output: base) => WITHDRAW
200       */
201      function _determineComputeType(
202          uint256 inputToken,
203:         uint256 outputToken

```
*GitHub*: [74](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L74-L77), [74](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L74-L77), [186](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L186-L189), [194](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L194-L202), [194](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L194-L203)

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

/// @audit Missing '@param amount'
12:      function withdraw(uint256 amount) external payable;

/// @audit Missing '@param ocean_'
/// @audit Missing '@param primitive_'
82       /**
83        * @notice only initializing the immutables, mappings & approves tokens
84        */
85:      constructor(address ocean_, address primitive_) OceanAdapter(ocean_, primitive_) {

/// @audit Missing '@param tokenAddress'
238      /**
239       * @dev Approves token to be spent by the Ocean and the Curve pool
240       */
241:     function _approveToken(address tokenAddress) private {

/// @audit Missing '@param tokenAddress'
246      /**
247       * @dev fetches underlying token balances
248       */
249:     function _getBalance(address tokenAddress) internal view returns (uint256 balance) {

/// @audit Missing '@param inputToken'
257      /**
258       * @dev Uses the inputToken and outputToken to determine the ComputeType
259       *  (input: xToken, output: yToken) | (input: yToken, output: xToken) => SWAP
260       *  base := xToken | yToken
261       *  (input: base, output: lpToken) => DEPOSIT
262       *  (input: lpToken, output: base) => WITHDRAW
263       */
264      function _determineComputeType(
265:         uint256 inputToken,

/// @audit Missing '@param outputToken'
257      /**
258       * @dev Uses the inputToken and outputToken to determine the ComputeType
259       *  (input: xToken, output: yToken) | (input: yToken, output: xToken) => SWAP
260       *  base := xToken | yToken
261       *  (input: base, output: lpToken) => DEPOSIT
262       *  (input: lpToken, output: base) => WITHDRAW
263       */
264      function _determineComputeType(
265          uint256 inputToken,
266:         uint256 outputToken

```
*GitHub*: [12](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L12-L12), [82](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L82-L85), [82](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L82-L85), [238](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L238-L241), [246](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L246-L249), [257](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L257-L265), [257](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L257-L266)

```solidity
File: src/adapters/OceanAdapter.sol

/// @audit Missing '@param ocean_'
/// @audit Missing '@param primitive_'
31       /// @notice only initializing the immutables
32:      constructor(address ocean_, address primitive_) {

/// @audit Missing '@param inputToken'
78       /**
79        * @notice Not implemented for this primitive
80        */
81       function computeInputAmount(
82:          uint256 inputToken,

/// @audit Missing '@param outputToken'
78       /**
79        * @notice Not implemented for this primitive
80        */
81       function computeInputAmount(
82           uint256 inputToken,
83:          uint256 outputToken,

/// @audit Missing '@param outputAmount'
78       /**
79        * @notice Not implemented for this primitive
80        */
81       function computeInputAmount(
82           uint256 inputToken,
83           uint256 outputToken,
84:          uint256 outputAmount,

/// @audit Missing '@param userAddress'
78       /**
79        * @notice Not implemented for this primitive
80        */
81       function computeInputAmount(
82           uint256 inputToken,
83           uint256 outputToken,
84           uint256 outputAmount,
85:          address userAddress,

/// @audit Missing '@param maximumInputAmount'
78       /**
79        * @notice Not implemented for this primitive
80        */
81       function computeInputAmount(
82           uint256 inputToken,
83           uint256 outputToken,
84           uint256 outputAmount,
85           address userAddress,
86:          bytes32 maximumInputAmount

/// @audit Missing '@param token'
/// @audit Missing '@param interactionType'
96       /**
97        * @notice used to fetch the Ocean interaction ID
98        */
99:      function _fetchInteractionId(address token, uint256 interactionType) internal pure returns (bytes32) {

/// @audit Missing '@param tokenAddress'
/// @audit Missing '@param tokenId'
105      /**
106       * @notice calculates Ocean ID for a underlying token
107       */
108:     function _calculateOceanId(address tokenAddress, uint256 tokenId) internal pure returns (uint256) {

/// @audit Missing '@param tokenId'
121      /**
122       * @notice returning 0 here since this primitive should not have any tokens
123       */
124:     function getTokenSupply(uint256 tokenId) external view override returns (uint256) {

/// @audit Missing '@param inputToken'
161      function primitiveOutputAmount(
162:         uint256 inputToken,

/// @audit Missing '@param outputToken'
161      function primitiveOutputAmount(
162          uint256 inputToken,
163:         uint256 outputToken,

/// @audit Missing '@param inputAmount'
161      function primitiveOutputAmount(
162          uint256 inputToken,
163          uint256 outputToken,
164:         uint256 inputAmount,

/// @audit Missing '@param metadata'
161      function primitiveOutputAmount(
162          uint256 inputToken,
163          uint256 outputToken,
164          uint256 inputAmount,
165:         bytes32 metadata

/// @audit Missing '@param tokenId'
/// @audit Missing '@param amount'
171:     function wrapToken(uint256 tokenId, uint256 amount) internal virtual;

/// @audit Missing '@param tokenId'
/// @audit Missing '@param amount'
173:     function unwrapToken(uint256 tokenId, uint256 amount) internal virtual;

```
*GitHub*: [31](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L31-L32), [31](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L31-L32), [78](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L78-L82), [78](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L78-L83), [78](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L78-L84), [78](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L78-L85), [78](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L78-L86), [96](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L96-L99), [96](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L96-L99), [105](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L105-L108), [105](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L105-L108), [121](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L121-L124), [161](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L161-L162), [161](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L161-L163), [161](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L161-L164), [161](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L161-L165), [171](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L171-L171), [171](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L171-L171), [173](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L173-L173), [173](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L173-L173)

```solidity
File: src/ocean/Ocean.sol

/// @audit Missing '@param uri_'
162      /**
163       * @dev Creates custom ERC-1155 with passed uri_, sets DAO address, and
164       *  initializes ERC-1155 transfer guard.
165       * @notice initializes the fee divisor to uint256 max, which results in
166       *  a fee of zero unless unwrapAmount == type(uint256).max, in which
167       *  case the fee is one part in 1.16 * 10^77.
168       */
169:     constructor(string memory uri_) OceanERC1155(uri_) {

/// @audit Missing '@param interfaceId'
301      /**
302       * @dev This callback is part of IERC1155Receiver, which we must implement
303       *  to wrap ERC-1155 tokens.
304       */
305:     function supportsInterface(bytes4 interfaceId) public view virtual override(OceanERC1155, IERC165) returns (bool) {

/// @audit Missing '@param outputToken'
810       *  derived from the contract address and a tokenId of 0.
811       * @notice Token amounts are normalized to 18 decimal places.
812       * @dev This means that to wrap 5 units of token A, which has 6 decimals,
813       *  and 5 units of token B, which has 18 decimals, the user would specify
814       *  5 * 10**18 for both token A and B.
815       * @param tokenAddress address of the ERC-20 token
816       * @param amount amount of the ERC-20 token to be wrapped, in terms of
817       *  18-decimal fixed point
818       * @param userAddress the address of the user who is wrapping the token
819       */
820:     function _erc20Wrap(address tokenAddress, uint256 amount, address userAddress, uint256 outputToken) private {

/// @audit Missing '@param inputToken'
854       *  the user would specify 5 * 10**18 for both token A and B.
855       * @dev If the fee is 1 basis point, the user would specify
856       *  5000500050005000500
857       *  This value was found by solving for x in the equation:
858       *      x - Floor[x * (1/10000)] = 5000000000000000000
859       * @param tokenAddress address of the ERC-20 token
860       * @param amount amount of the ERC-20 token to be unwrapped, in terms of
861       *  18-decimal fixed point
862       * @param userAddress the address of the user who is unwrapping the token
863       */
864:     function _erc20Unwrap(address tokenAddress, uint256 amount, address userAddress, uint256 inputToken) private {

/// @audit Missing '@param oceanId'
882      /**
883       * @dev wrap an ERC-721 NFT into the Ocean. The Ocean ID is derived from
884       *  tokenAddress and tokenId.
885       * @param tokenAddress address of the ERC-721 contract
886       * @param tokenId ID of the NFT on the ERC-721 ledger
887       * @param userAddress the address of the user who is wrapping the NFT
888       */
889:     function _erc721Wrap(address tokenAddress, uint256 tokenId, address userAddress, uint256 oceanId) private {

/// @audit Missing '@param oceanId'
896      /**
897       * @dev Unwrap an ERC-721 NFT out of the Ocean. The Ocean ID is derived
898       *  from tokenAddress and tokenId.
899       * @param tokenAddress address of the ERC-721 contract
900       * @param tokenId ID of the NFT on the ERC-721 ledger
901       * @param userAddress the address of the user who is unwrapping the NFT
902       */
903:     function _erc721Unwrap(address tokenAddress, uint256 tokenId, address userAddress, uint256 oceanId) private {

/// @audit Missing '@param oceanId'
915       * @param tokenAddress address of the ERC-1155 contract
916       * @param tokenId ID of the token on the ERC-1155 ledger
917       * @param amount the amount of the token being wrapped.
918       * @param userAddress the address of the user who is wrapping the token
919       */
920      function _erc1155Wrap(
921          address tokenAddress,
922          uint256 tokenId,
923          uint256 amount,
924          address userAddress,
925:         uint256 oceanId

/// @audit Missing '@param oceanId'
950       * @param tokenAddress address of the ERC-1155 contract
951       * @param tokenId ID of the token on the ERC-1155 ledger
952       * @param amount the amount of the token being wrapped.
953       * @param userAddress the address of the user who is wrapping the token
954       */
955      function _erc1155Unwrap(
956          address tokenAddress,
957          uint256 tokenId,
958          uint256 amount,
959          address userAddress,
960:         uint256 oceanId

/// @audit Missing '@param primitive'
/// @audit Missing '@param inputToken'
/// @audit Missing '@param inputAmount'
994       *  When the ocean receives a balanceOf() call, this call has its own
995       *  memory space. The ocean cannot reach down through the call stack to
996       *  get a delta stored in the memory of an earlier call.
997       *      primitive -> ocean.balanceOf(address(this), token)  [mem_space_2]
998       *      ocean -> primitive.computeOutputAmount()          [mem_space_1]
999       *      EOA -> ocean.doMultipleInteractions()               [mem_space_0]
1000      *  Because we have no way of maintaining coherence between dirty memory
1001      *  and stale storage, our only option is to always have up-to-date values
1002      *  in storage.
1003      */
1004:    function _increaseBalanceOfPrimitive(address primitive, uint256 inputToken, uint256 inputAmount) internal {

/// @audit Missing '@param primitive'
/// @audit Missing '@param outputToken'
/// @audit Missing '@param outputAmount'
1028      *  When the ocean receives a balanceOf() call, this call has its own
1029      *  memory space. The ocean cannot reach down through the call stack to
1030      *  get a delta stored in the memory of an earlier call.
1031      *      primitive -> ocean.balanceOf(address(this), token)  [mem_space_2]
1032      *      ocean -> primitive.computeOutputAmount()          [mem_space_1]
1033      *      EOA -> ocean.doMultipleInteractions()               [mem_space_0]
1034      *  Because we have no way of maintaining coherence between dirty memory
1035      *  and stale storage, our only option is to always have up-to-date values
1036      *  in storage.
1037      */
1038:    function _decreaseBalanceOfPrimitive(address primitive, uint256 outputToken, uint256 outputAmount) internal {

/// @audit Missing '@param oceanId'
/// @audit Missing '@param amount'
1162     /**
1163      * @dev Updates the DAO's balance of a token when the fee is assessed.
1164      * @dev We only call the mint function when the fee amount is non-zero.
1165      */
1166:    function _grantFeeToOcean(uint256 oceanId, uint256 amount) private {

```
*GitHub*: [162](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L162-L169), [301](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L301-L305), [810](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L810-L820), [854](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L854-L864), [882](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L882-L889), [896](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L896-L903), [915](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L915-L925), [950](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L950-L960), [994](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L994-L1004), [994](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L994-L1004), [994](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L994-L1004), [1028](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L1028-L1038), [1028](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L1028-L1038), [1028](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L1028-L1038), [1162](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L1162-L1166), [1162](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L1162-L1166)



### [N&#x2011;16] NatSpec: Function `@return` tag is missing


*There are 41 instances of this issue:*

```solidity
File: src/adapters/Curve2PoolAdapter.sol

/// @audit Missing '@return outputAmount'
135      /**
136       * @dev swaps/add liquidity/remove liquidity from Curve 2pool
137       * @param inputToken The user is giving this token to the pool
138       * @param outputToken The pool is giving this token to the user
139       * @param inputAmount The amount of the inputToken the user is giving to the pool
140       * @param minimumOutputAmount The minimum amount of tokens expected back after the exchange
141       */
142      function primitiveOutputAmount(
143          uint256 inputToken,
144          uint256 outputToken,
145          uint256 inputAmount,
146          bytes32 minimumOutputAmount
147      )
148          internal
149          override
150          returns (uint256 outputAmount)
151:     {

/// @audit Missing '@return computeType'
194      /**
195       * @dev Uses the inputToken and outputToken to determine the ComputeType
196       *  (input: xToken, output: yToken) | (input: yToken, output: xToken) => SWAP
197       *  base := xToken | yToken
198       *  (input: base, output: lpToken) => DEPOSIT
199       *  (input: lpToken, output: base) => WITHDRAW
200       */
201      function _determineComputeType(
202          uint256 inputToken,
203          uint256 outputToken
204      )
205          private
206          view
207          returns (ComputeType computeType)
208:     {

```
*GitHub*: [135](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L135-L151), [194](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L194-L208)

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

/// @audit Missing '@return outputAmount'
171      /**
172       * @dev swaps/add liquidity/remove liquidity from Curve Tricrypto Pool
173       * @param inputToken The user is giving this token to the pool
174       * @param outputToken The pool is giving this token to the user
175       * @param inputAmount The amount of the inputToken the user is giving to the pool
176       * @param minimumOutputAmount The minimum amount of tokens expected back after the exchange
177       */
178      function primitiveOutputAmount(
179          uint256 inputToken,
180          uint256 outputToken,
181          uint256 inputAmount,
182          bytes32 minimumOutputAmount
183      )
184          internal
185          override
186          returns (uint256 outputAmount)
187:     {

/// @audit Missing '@return balance'
246      /**
247       * @dev fetches underlying token balances
248       */
249:     function _getBalance(address tokenAddress) internal view returns (uint256 balance) {

/// @audit Missing '@return computeType'
257      /**
258       * @dev Uses the inputToken and outputToken to determine the ComputeType
259       *  (input: xToken, output: yToken) | (input: yToken, output: xToken) => SWAP
260       *  base := xToken | yToken
261       *  (input: base, output: lpToken) => DEPOSIT
262       *  (input: lpToken, output: base) => WITHDRAW
263       */
264      function _determineComputeType(
265          uint256 inputToken,
266          uint256 outputToken
267      )
268          private
269          view
270          returns (ComputeType computeType)
271:     {

```
*GitHub*: [171](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L171-L187), [246](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L246-L249), [257](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L257-L271)

```solidity
File: src/adapters/OceanAdapter.sol

/// @audit Missing '@return outputAmount'
45        *  do the accounting.  One of the token amounts is chosen by the user, and
46        *  the other amount is chosen by the primitive.  When computeOutputAmount is
47        *  called, the user provides the inputAmount, and the primitive uses this to
48        *  compute the outputAmount
49        * @param inputToken The user is giving this token to the primitive
50        * @param outputToken The primitive is giving this token to the user
51        * @param inputAmount The amount of the inputToken the user is giving to the primitive
52        * @param metadata a bytes32 value that the user provides the Ocean
53        * @dev the unused param is an address field called userAddress
54        */
55       function computeOutputAmount(
56           uint256 inputToken,
57           uint256 outputToken,
58           uint256 inputAmount,
59           address,
60           bytes32 metadata
61       )
62           external
63           override
64           onlyOcean
65           returns (uint256 outputAmount)
66:      {

/// @audit Missing '@return inputAmount'
78       /**
79        * @notice Not implemented for this primitive
80        */
81       function computeInputAmount(
82           uint256 inputToken,
83           uint256 outputToken,
84           uint256 outputAmount,
85           address userAddress,
86           bytes32 maximumInputAmount
87       )
88           external
89           override
90           onlyOcean
91           returns (uint256 inputAmount)
92:      {

/// @audit Missing '@return  '
96       /**
97        * @notice used to fetch the Ocean interaction ID
98        */
99:      function _fetchInteractionId(address token, uint256 interactionType) internal pure returns (bytes32) {

/// @audit Missing '@return  '
105      /**
106       * @notice calculates Ocean ID for a underlying token
107       */
108:     function _calculateOceanId(address tokenAddress, uint256 tokenId) internal pure returns (uint256) {

/// @audit Missing '@return  '
121      /**
122       * @notice returning 0 here since this primitive should not have any tokens
123       */
124:     function getTokenSupply(uint256 tokenId) external view override returns (uint256) {

/// @audit Missing '@return outputAmount'
161      function primitiveOutputAmount(
162          uint256 inputToken,
163          uint256 outputToken,
164          uint256 inputAmount,
165          bytes32 metadata
166      )
167          internal
168          virtual
169:         returns (uint256 outputAmount);

```
*GitHub*: [45](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L45-L66), [78](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L78-L92), [96](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L96-L99), [105](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L105-L108), [121](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L121-L124), [161](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L161-L169)

```solidity
File: src/ocean/Ocean.sol

/// @audit Missing '@return burnAmount'
/// @audit Missing '@return mintId'
/// @audit Missing '@return mintAmount'
/// @audit Missing '@return burnId'
203      /**
204       * @notice Execute interactions `interaction`
205       * @notice Does not need ids because a single interaction does not require
206       *  the accounting system
207       * @dev call to _doInteraction() binds msg.sender to userAddress
208       * @param interaction Executed to produce a set of balance updates
209       */
210      function doInteraction(Interaction calldata interaction)
211          external
212          payable
213          override
214          returns (uint256 burnId, uint256 burnAmount, uint256 mintId, uint256 mintAmount)
215:     {

/// @audit Missing '@return burnIds'
/// @audit Missing '@return burnAmounts'
/// @audit Missing '@return mintIds'
/// @audit Missing '@return mintAmounts'
220      /**
221       * @notice Execute interactions `interactions` with tokens `ids`
222       * @notice ids must include all tokens invoked during the transaction
223       * @notice ids are used for memory allocation in the intra-transaction
224       *  accounting system.
225       * @dev call to _doMultipleInteractions() binds msg.sender to userAddress
226       * @param interactions Executed to produce a set of balance updates
227       * @param ids Ocean IDs of the tokens invoked by the interactions.
228       */
229      function doMultipleInteractions(
230          Interaction[] calldata interactions,
231          uint256[] calldata ids
232      )
233          external
234          payable
235          override
236          returns (
237              uint256[] memory burnIds,
238              uint256[] memory burnAmounts,
239              uint256[] memory mintIds,
240              uint256[] memory mintAmounts
241          )
242:     {

/// @audit Missing '@return burnId'
/// @audit Missing '@return burnAmount'
/// @audit Missing '@return mintId'
/// @audit Missing '@return mintAmount'
247      /**
248       * @notice Execute interactions `interactions` on behalf of `userAddress`
249       * @notice Does not need ids because a single interaction does not require
250       *  the overhead of the intra-transaction accounting system
251       * @dev MUST HAVE onlyApprovedForwarder modifer.
252       * @dev call to _doMultipleInteractions() forwards the userAddress
253       * @param interaction Executed to produce a set of balance updates
254       * @param userAddress interactions are executed on behalf of this address
255       */
256      function forwardedDoInteraction(
257          Interaction calldata interaction,
258          address userAddress
259      )
260          external
261          payable
262          override
263          onlyApprovedForwarder(userAddress)
264          returns (uint256 burnId, uint256 burnAmount, uint256 mintId, uint256 mintAmount)
265:     {

/// @audit Missing '@return mintIds'
/// @audit Missing '@return mintAmounts'
/// @audit Missing '@return burnIds'
/// @audit Missing '@return burnAmounts'
271       * @notice Execute interactions `interactions` with tokens `ids` on behalf of `userAddress`
272       * @notice ids must include all tokens invoked during the transaction
273       * @notice ids are used for memory allocation in the intra-transaction
274       *  accounting system.
275       * @dev MUST HAVE onlyApprovedForwarder modifer.
276       * @dev call to _doMultipleInteractions() forwards the userAddress
277       * @param interactions Executed to produce a set of balance updates
278       * @param ids Ocean IDs of the tokens invoked by the interactions.
279       * @param userAddress interactions are executed on behalf of this address
280       */
281      function forwardedDoMultipleInteractions(
282          Interaction[] calldata interactions,
283          uint256[] calldata ids,
284          address userAddress
285      )
286          external
287          payable
288          override
289          onlyApprovedForwarder(userAddress)
290          returns (
291              uint256[] memory burnIds,
292              uint256[] memory burnAmounts,
293              uint256[] memory mintIds,
294              uint256[] memory mintAmounts
295          )
296:     {

/// @audit Missing '@return  '
301      /**
302       * @dev This callback is part of IERC1155Receiver, which we must implement
303       *  to wrap ERC-1155 tokens.
304       */
305:     function supportsInterface(bytes4 interfaceId) public view virtual override(OceanERC1155, IERC165) returns (bool) {

/// @audit Missing '@return  '
309:     function onERC721Received(address, address, uint256, bytes calldata) external view override returns (bytes4) {

/// @audit Missing '@return  '
317      /**
318       * @dev This callback is part of IERC1155Receiver, which we must implement
319       *  to wrap ERC-1155 tokens.
320       * @dev The Ocean only accepts ERC1155 transfers initiated by the Ocean
321       *  while executing interactions.
322       * @dev We don't revert, prefering to let the external contract
323       *  decide what it wants to do when safeTransfer is called on a contract
324       *  that does not return the expected selector.
325       */
326      function onERC1155Received(
327          address,
328          address,
329          uint256,
330          uint256,
331          bytes calldata
332      )
333          external
334          view
335          override
336          returns (bytes4)
337:     {

/// @audit Missing '@return  '
345      /**
346       * @dev This callback is part of IERC1155Receiver, which we must implement
347       *  to wrap ERC-1155 tokens.
348       * @dev The Ocean never initiates ERC1155 Batch Transfers.
349       * @dev We don't revert, prefering to let the external contract
350       *  decide what it wants to do when safeTransfer is called on a contract
351       *  that does not return the expected selector.
352       */
353      function onERC1155BatchReceived(
354          address,
355          address,
356          uint256[] calldata,
357          uint256[] calldata,
358          bytes calldata
359      )
360          external
361          pure
362          override
363          returns (bytes4)
364:     {

/// @audit Missing '@return inputToken'
/// @audit Missing '@return inputAmount'
/// @audit Missing '@return outputToken'
/// @audit Missing '@return outputAmount'
370       *  interactions
371       * @dev the external functions that pass through their arguments to this
372       *  function have more information about the arguments.
373       * @param interaction the current interaction passed by the caller
374       * @param userAddress In the case of a forwarded interaction, this value
375       *  is passed by the caller, and this value must be validated against the
376       *  approvals set on this address and the caller's address (`msg.sender`)
377       *  In the case of a non-forwarded interaction, this value is the caller's
378       *  address.
379       */
380      function _doInteraction(
381          Interaction calldata interaction,
382          address userAddress
383      )
384          internal
385          returns (uint256 inputToken, uint256 inputAmount, uint256 outputToken, uint256 outputAmount)
386:     {

/// @audit Missing '@return burnIds'
/// @audit Missing '@return burnAmounts'
/// @audit Missing '@return mintIds'
/// @audit Missing '@return mintAmounts'
435       * @dev the external functions that pass through their arguments to this
436       *  function have more information about the arguments.
437       * @param interactions The interactions passed by the caller
438       * @param ids the ids passed by the caller
439       * @param userAddress In the case of a forwarded interaction, this value
440       *  is passed by the caller, and this value must be validated against the
441       *  approvals set on this address and the caller's address (`msg.sender`)
442       *  In the case of a non-forwarded interaction, this value is the caller's
443       *  address.
444       */
445      function _doMultipleInteractions(
446          Interaction[] calldata interactions,
447          uint256[] calldata ids,
448          address userAddress
449      )
450          internal
451          returns (
452              uint256[] memory burnIds,
453              uint256[] memory burnAmounts,
454              uint256[] memory mintIds,
455              uint256[] memory mintAmounts
456          )
457:     {

/// @audit Missing '@return interactionType'
/// @audit Missing '@return externalContract'
676      /**
677       * @param interaction the interaction
678       * @dev the first byte contains the interactionType
679       * @dev the next eleven bytes are IGNORED
680       * @dev the final twenty bytes are the address targeted by the interaction
681       */
682      function _unpackInteractionTypeAndAddress(Interaction memory interaction)
683          internal
684          pure
685          returns (InteractionType interactionType, address externalContract)
686:     {

```
*GitHub*: [203](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L203-L215), [203](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L203-L215), [203](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L203-L215), [203](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L203-L215), [220](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L220-L242), [220](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L220-L242), [220](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L220-L242), [220](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L220-L242), [247](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L247-L265), [247](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L247-L265), [247](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L247-L265), [247](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L247-L265), [271](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L271-L296), [271](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L271-L296), [271](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L271-L296), [271](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L271-L296), [301](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L301-L305), [309](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L309-L309), [317](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L317-L337), [345](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L345-L364), [370](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L370-L386), [370](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L370-L386), [370](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L370-L386), [370](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L370-L386), [435](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L435-L457), [435](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L435-L457), [435](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L435-L457), [435](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L435-L457), [676](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L676-L686), [676](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L676-L686)



### [N&#x2011;17] NatSpec: Function declarations should have `@notice` tags
`@notice` is used to explain to end users what the function does, and the compiler interprets `///` or `/**` comments (but not `//` or `/*`) as this tag if one wasn't explicitly provided

*There are 4 instances of this issue:*

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

11:      function deposit() external payable;

12:      function withdraw(uint256 amount) external payable;

291:     fallback() external payable { }

```
*GitHub*: [11](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L11-L11), [12](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L12-L12), [291](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L291-L291)

```solidity
File: src/ocean/Ocean.sol

309:     function onERC721Received(address, address, uint256, bytes calldata) external view override returns (bytes4) {

```
*GitHub*: [309](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L309-L309)



### [N&#x2011;18] NatSpec: Non-public state variable declarations should use `@dev` tags
i.e. `@dev` [tags](https://docs.soliditylang.org/en/latest/natspec-format.html#tags). Note that since they're non-public, `@notice` is not the right tag to use.

*There are 6 instances of this issue:*

```solidity
File: src/adapters/Curve2PoolAdapter.sol

65:      mapping(uint256 => int128) indexOf;

68:      mapping(uint256 => uint8) decimals;

```
*GitHub*: [65](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L65-L65), [68](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L68-L68)

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

73:      mapping(uint256 => uint256) indexOf;

76:      mapping(uint256 => uint8) decimals;

```
*GitHub*: [73](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L73-L73), [76](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L76-L76)

```solidity
File: src/adapters/OceanAdapter.sol

16:      uint8 constant NORMALIZED_DECIMALS = 18;

```
*GitHub*: [16](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L16-L16)

```solidity
File: src/ocean/Ocean.sol

102:     uint256 constant GET_BALANCE_DELTA = type(uint256).max;

```
*GitHub*: [102](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L102-L102)



### [N&#x2011;19] Style guide: Extraneous whitespace
See the [whitespace](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#whitespace-in-expressions) section of the Solidity Style Guide

*There are 16 instances of this issue:*

```solidity
File: src/adapters/Curve2PoolAdapter.sol

6:    import { IERC20Metadata } from "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";

```
*GitHub*: [6](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L6)

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

6:    import { IERC20Metadata } from "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";

```
*GitHub*: [6](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L6)

```solidity
File: src/adapters/OceanAdapter.sol

6:    import { IERC20 } from "@openzeppelin/contracts/token/ERC20/IERC20.sol";

```
*GitHub*: [6](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L6)

```solidity
File: src/ocean/Ocean.sol

9:    import { IERC20Metadata } from "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";

10:   import { IERC20 } from "@openzeppelin/contracts/token/ERC20/IERC20.sol";

11:   import { IERC165 } from "@openzeppelin/contracts/utils/introspection/IERC165.sol";

12:   import { IERC721 } from "@openzeppelin/contracts/token/ERC721/IERC721.sol";

13:   import { IERC1155 } from "@openzeppelin/contracts/token/ERC1155/IERC1155.sol";

14:   import { IERC1155Receiver } from "@openzeppelin/contracts/token/ERC1155/IERC1155Receiver.sol";

15:   import { IERC721Receiver } from "@openzeppelin/contracts/token/ERC721/IERC721Receiver.sol";

18:   import { SafeERC20 } from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

21:   import { IOceanInteractions, Interaction, InteractionType } from "./Interactions.sol";

22:   import { IOceanFeeChange } from "./IOceanFeeChange.sol";

23:   import { IOceanPrimitive } from "./IOceanPrimitive.sol";

24:   import { BalanceDelta, LibBalanceDelta } from "./BalanceDelta.sol";

27:   import { OceanERC1155 } from "./OceanERC1155.sol";

```
*GitHub*: [9](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L9), [10](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L10), [11](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L11), [12](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L12), [13](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L13), [14](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L14), [15](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L15), [18](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L18), [21](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L21), [22](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L22), [23](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L23), [24](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L24), [27](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L27)



### [N&#x2011;20] Style guide: Non-`external`/`public` variable names should begin with an underscore
According to the Solidity Style Guide, non-`external`/`public` variable names should begin with an [underscore](https://docs.soliditylang.org/en/latest/style-guide.html#underscore-prefix-for-non-external-functions-and-variables)

*There are 4 instances of this issue:*

```solidity
File: src/adapters/Curve2PoolAdapter.sol

65:      mapping(uint256 => int128) indexOf;

68:      mapping(uint256 => uint8) decimals;

```
*GitHub*: [65](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L65-L65), [68](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L68-L68)

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

73:      mapping(uint256 => uint256) indexOf;

76:      mapping(uint256 => uint8) decimals;

```
*GitHub*: [73](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L73-L73), [76](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L76-L76)



### [N&#x2011;21] Unused function parameter
Comment out the variable name to suppress compiler warnings

*There are 6 instances of this issue:*

```solidity
File: src/adapters/OceanAdapter.sol

/// @audit outputAmount
/// @audit maximumInputAmount
/// @audit userAddress
/// @audit inputToken
/// @audit outputToken
81       function computeInputAmount(
82           uint256 inputToken,
83           uint256 outputToken,
84           uint256 outputAmount,
85           address userAddress,
86           bytes32 maximumInputAmount
87       )
88           external
89           override
90           onlyOcean
91           returns (uint256 inputAmount)
92:      {

/// @audit tokenId
124:     function getTokenSupply(uint256 tokenId) external view override returns (uint256) {

```
*GitHub*: [81](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L81-L92), [81](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L81-L92), [81](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L81-L92), [81](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L81-L92), [81](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L81-L92), [124](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L124-L124)



### [N&#x2011;22] Use of `override` is unnecessary
Starting with Solidity version [0.8.8](https://docs.soliditylang.org/en/v0.8.20/contracts.html#function-overriding), using the `override` keyword when the function solely overrides an interface function, and the function doesn't exist in multiple base contracts, is unnecessary.

*There are 18 instances of this issue:*

```solidity
File: src/adapters/Curve2PoolAdapter.sol

102:     function wrapToken(uint256 tokenId, uint256 amount) internal override {

121:     function unwrapToken(uint256 tokenId, uint256 amount) internal override {

142      function primitiveOutputAmount(
143          uint256 inputToken,
144          uint256 outputToken,
145          uint256 inputAmount,
146          bytes32 minimumOutputAmount
147      )
148          internal
149          override
150          returns (uint256 outputAmount)
151:     {

```
*GitHub*: [102](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L102-L102), [121](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L121-L121), [142](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/Curve2PoolAdapter.sol#L142-L151)

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

118:     function wrapToken(uint256 tokenId, uint256 amount) internal override {

147:     function unwrapToken(uint256 tokenId, uint256 amount) internal override {

178      function primitiveOutputAmount(
179          uint256 inputToken,
180          uint256 outputToken,
181          uint256 inputAmount,
182          bytes32 minimumOutputAmount
183      )
184          internal
185          override
186          returns (uint256 outputAmount)
187:     {

```
*GitHub*: [118](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L118-L118), [147](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L147-L147), [178](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/CurveTricryptoAdapter.sol#L178-L187)

```solidity
File: src/adapters/OceanAdapter.sol

55       function computeOutputAmount(
56           uint256 inputToken,
57           uint256 outputToken,
58           uint256 inputAmount,
59           address,
60           bytes32 metadata
61       )
62           external
63           override
64           onlyOcean
65           returns (uint256 outputAmount)
66:      {

81       function computeInputAmount(
82           uint256 inputToken,
83           uint256 outputToken,
84           uint256 outputAmount,
85           address userAddress,
86           bytes32 maximumInputAmount
87       )
88           external
89           override
90           onlyOcean
91           returns (uint256 inputAmount)
92:      {

124:     function getTokenSupply(uint256 tokenId) external view override returns (uint256) {

```
*GitHub*: [55](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L55-L66), [81](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L81-L92), [124](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/adapters/OceanAdapter.sol#L124-L124)

```solidity
File: src/ocean/Ocean.sol

196:     function changeUnwrapFee(uint256 nextUnwrapFeeDivisor) external override onlyOwner {

210      function doInteraction(Interaction calldata interaction)
211          external
212          payable
213          override
214          returns (uint256 burnId, uint256 burnAmount, uint256 mintId, uint256 mintAmount)
215:     {

229      function doMultipleInteractions(
230          Interaction[] calldata interactions,
231          uint256[] calldata ids
232      )
233          external
234          payable
235          override
236          returns (
237              uint256[] memory burnIds,
238              uint256[] memory burnAmounts,
239              uint256[] memory mintIds,
240              uint256[] memory mintAmounts
241          )
242:     {

256      function forwardedDoInteraction(
257          Interaction calldata interaction,
258          address userAddress
259      )
260          external
261          payable
262          override
263          onlyApprovedForwarder(userAddress)
264          returns (uint256 burnId, uint256 burnAmount, uint256 mintId, uint256 mintAmount)
265:     {

281      function forwardedDoMultipleInteractions(
282          Interaction[] calldata interactions,
283          uint256[] calldata ids,
284          address userAddress
285      )
286          external
287          payable
288          override
289          onlyApprovedForwarder(userAddress)
290          returns (
291              uint256[] memory burnIds,
292              uint256[] memory burnAmounts,
293              uint256[] memory mintIds,
294              uint256[] memory mintAmounts
295          )
296:     {

305:     function supportsInterface(bytes4 interfaceId) public view virtual override(OceanERC1155, IERC165) returns (bool) {

309:     function onERC721Received(address, address, uint256, bytes calldata) external view override returns (bytes4) {

326      function onERC1155Received(
327          address,
328          address,
329          uint256,
330          uint256,
331          bytes calldata
332      )
333          external
334          view
335          override
336          returns (bytes4)
337:     {

353      function onERC1155BatchReceived(
354          address,
355          address,
356          uint256[] calldata,
357          uint256[] calldata,
358          bytes calldata
359      )
360          external
361          pure
362          override
363          returns (bytes4)
364:     {

```
*GitHub*: [196](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L196-L196), [210](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L210-L215), [229](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L229-L242), [256](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L256-L265), [281](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L281-L296), [305](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L305-L305), [309](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L309-L309), [326](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L326-L337), [353](https://github.com/code-423n4/2023-11-shellprotocol/blob/2bcf608ab18978c4c061817dd92d6aa2a868ac06/src/ocean/Ocean.sol#L353-L364)
