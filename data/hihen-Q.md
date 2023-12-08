# QA Report

## Low Issues

### [L-01] Use Ownable2Step instead of Ownable

[`Ownable2Step`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/3d7a93876a2e5e1d7fe29b5a0e96e222afdc4cfa/contracts/access/Ownable2Step.sol#L31-L56) and [`Ownable2StepUpgradeable`](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/25aabd286e002a1526c345c8db259d57bdf0ad28/contracts/access/Ownable2StepUpgradeable.sol#L47-L63) prevent the contract ownership from mistakenly being transferred to an address that cannot handle it (e.g. due to a typo in the address), by requiring that the recipient of the owner permissions actively accept via a contract call of its own.

There is 1 instance:

- *Ocean.sol* ( [79](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L79) ):

```solidity
/// @audit `Ocean` is Ownable
79: contract Ocean is IOceanInteractions, IOceanFeeChange, OceanERC1155, IERC721Receiver, IERC1155Receiver {
```

### [L-02] Constructor / initialization function lacks parameter validation

Constructors and initialization functions play a critical role in contracts by setting important initial states when the contract is first deployed before the system starts. The parameters passed to the constructor and initialization functions directly affect the behavior of the contract / protocol. If incorrect parameters are provided, the system may fail to run, behave abnormally, be unstable, or lack security. Therefore, it's crucial to carefully check each parameter in the constructor and initialization functions. If an exception is found, the transaction should be rolled back.

There are 7 instances:

- *Curve2PoolAdapter.sol* ( [77](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L77) ):

```solidity
/// @audit `ocean_`
/// @audit `primitive_`
77:     constructor(address ocean_, address primitive_) OceanAdapter(ocean_, primitive_) {
```

- *CurveTricryptoAdapter.sol* ( [85](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L85) ):

```solidity
/// @audit `ocean_`
/// @audit `primitive_`
85:     constructor(address ocean_, address primitive_) OceanAdapter(ocean_, primitive_) {
```

- *OceanAdapter.sol* ( [32](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L32) ):

```solidity
/// @audit `ocean_`
/// @audit `primitive_`
32:     constructor(address ocean_, address primitive_) {
```

- *Ocean.sol* ( [169](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L169) ):

```solidity
/// @audit `uri_`
169:     constructor(string memory uri_) OceanERC1155(uri_) {
```

### [L-03] Contracts are vulnerable to fee-on-transfer accounting-related issues

Some tokens take a transfer fee (e.g. STA, PAXG), some do not currently charge a fee but may do so in the future (e.g. USDT, USDC).
The functions below transfer funds from the caller to the receiver via `transferFrom()`, but do not ensure that the actual number of tokens received is the same as the input amount to the transfer. If the token is a fee-on-transfer token, the balance after the transfer will be smaller than expected, leading to accounting issues. Even if there are checks later, related to a secondary transfer, an attacker may be able to use latent funds (e.g. mistakenly sent by another user) in order to get a free credit.
One way to solve this problem is to measure the balance before and after the transfer, and use the difference as the amount, rather than the stated amount.

There are 2 instances:

- *Ocean.sol* ( [836](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L836), [875](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L875) ):

```solidity
836:             SafeERC20.safeTransferFrom(IERC20(tokenAddress), userAddress, address(this), transferAmount);

875:             SafeERC20.safeTransfer(IERC20(tokenAddress), userAddress, transferAmount);
```

### [L-04] External function calls within loops

Calling external functions within loops can easily result in insufficient gas. This greatly increases the likelihood of transaction failures, DOS attacks, and other unexpected actions. It is recommended to limit the number of loops within loops that call external functions, and to limit the gas line for each external call.

There is 1 instance:

- *Ocean.sol* ( [522-524](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L522-L524) ):

```solidity
/// @audit Calling `_executeInteraction()` within `for` loop, it will trigger an external call - `computeOutputAmount()`
522:                 _executeInteraction(
523:                     interaction, interactionType, externalContract, specifiedToken, specifiedAmount, userAddress_
524:                 );
```

### [L-05] Contracts are vulnerable to rebasing accounting-related issues

Rebasing tokens are tokens that have each holder's `balanceof()` increase over time. Aave aTokens are an example of such tokens. If rebasing tokens are used, rewards accrue to the contract holding the tokens, and cannot be withdrawn by the original depositor. To address the issue, track 'shares' deposited on a pro-rata basis, and let shares be redeemed for their proportion of the current balance at the time of the withdrawal.

There are 2 instances:

- *Ocean.sol* ( [820](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L820), [864](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L864) ):

```solidity
820:     function _erc20Wrap(address tokenAddress, uint256 amount, address userAddress, uint256 outputToken) private {

864:     function _erc20Unwrap(address tokenAddress, uint256 amount, address userAddress, uint256 inputToken) private {
```

## Non Critical Issues

### [N-01] Too long functions should be refactored

Functions with too many lines are difficult to understand. It is recommended to refactor complex functions into multiple shorter and easier to understand functions.

There are 4 instances:

- *CurveTricryptoAdapter.sol* ( [178](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L178) ):

```solidity
/// @audit 59 lines
178:     function primitiveOutputAmount(
```

- *Ocean.sol* ( [380](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L380), [445](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L445), [597](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L597) ):

```solidity
/// @audit 51 lines
380:     function _doInteraction(

/// @audit 129 lines
445:     function _doMultipleInteractions(

/// @audit 78 lines
597:     function _executeInteraction(
```

### [N-02] Use of `override` is unnecessary

[Starting from Solidity 0.8.8](https://docs.soliditylang.org/en/v0.8.20/contracts.html#function-overriding), the override keyword is not required when overriding an interface function, except for the case where the function is defined in multiple bases.

<details>
<summary>There are 11 instances (click to show):</summary>

- *OceanAdapter.sol* ( [55-65](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L55-L65), [81-91](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L81-L91), [124](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L124) ):

```solidity
55:     function computeOutputAmount(
56:         uint256 inputToken,
57:         uint256 outputToken,
58:         uint256 inputAmount,
59:         address,
60:         bytes32 metadata
61:     )
62:         external
63:         override
64:         onlyOcean
65:         returns (uint256 outputAmount)

81:     function computeInputAmount(
82:         uint256 inputToken,
83:         uint256 outputToken,
84:         uint256 outputAmount,
85:         address userAddress,
86:         bytes32 maximumInputAmount
87:     )
88:         external
89:         override
90:         onlyOcean
91:         returns (uint256 inputAmount)

124:     function getTokenSupply(uint256 tokenId) external view override returns (uint256) {
```

- *Ocean.sol* ( [196](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L196), [210-214](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L210-L214), [229-241](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L229-L241), [256-264](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L256-L264), [281-295](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L281-L295), [309](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L309), [326-336](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L326-L336), [353-363](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L353-L363) ):

```solidity
196:     function changeUnwrapFee(uint256 nextUnwrapFeeDivisor) external override onlyOwner {

210:     function doInteraction(Interaction calldata interaction)
211:         external
212:         payable
213:         override
214:         returns (uint256 burnId, uint256 burnAmount, uint256 mintId, uint256 mintAmount)

229:     function doMultipleInteractions(
230:         Interaction[] calldata interactions,
231:         uint256[] calldata ids
232:     )
233:         external
234:         payable
235:         override
236:         returns (
237:             uint256[] memory burnIds,
238:             uint256[] memory burnAmounts,
239:             uint256[] memory mintIds,
240:             uint256[] memory mintAmounts
241:         )

256:     function forwardedDoInteraction(
257:         Interaction calldata interaction,
258:         address userAddress
259:     )
260:         external
261:         payable
262:         override
263:         onlyApprovedForwarder(userAddress)
264:         returns (uint256 burnId, uint256 burnAmount, uint256 mintId, uint256 mintAmount)

281:     function forwardedDoMultipleInteractions(
282:         Interaction[] calldata interactions,
283:         uint256[] calldata ids,
284:         address userAddress
285:     )
286:         external
287:         payable
288:         override
289:         onlyApprovedForwarder(userAddress)
290:         returns (
291:             uint256[] memory burnIds,
292:             uint256[] memory burnAmounts,
293:             uint256[] memory mintIds,
294:             uint256[] memory mintAmounts
295:         )

309:     function onERC721Received(address, address, uint256, bytes calldata) external view override returns (bytes4) {

326:     function onERC1155Received(
327:         address,
328:         address,
329:         uint256,
330:         uint256,
331:         bytes calldata
332:     )
333:         external
334:         view
335:         override
336:         returns (bytes4)

353:     function onERC1155BatchReceived(
354:         address,
355:         address,
356:         uint256[] calldata,
357:         uint256[] calldata,
358:         bytes calldata
359:     )
360:         external
361:         pure
362:         override
363:         returns (bytes4)
```

</details>

### [N-03] Contracts should each be defined in separate files

Keeping each contract in a separate file makes it easier to work with multiple people, makes the code easier to maintain, and is a common practice on most projects. The following files each contains more than one contract/library/interface.

There is 1 instance:

- [*CurveTricryptoAdapter.sol*](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol)

### [N-04] Function state mutability can be restricted to pure

Function state mutability can be restricted to pure

There is 1 instance:

- *OceanAdapter.sol* ( [124](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L124) ):

```solidity
124:     function getTokenSupply(uint256 tokenId) external view override returns (uint256) {
```

### [N-05] Contract implements interface without extending the interface

Not extending the interface may lead to the wrong function signature being used, leading to unexpected behavior. If the interface is in fact being implemented, use the `override` keyword to indicate that fact.

There is 1 instance:

- *OceanAdapter.sol* ( [14-15](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L14-L15) ):

```solidity
/// @audit IERC1155Receiver
14: abstract contract OceanAdapter is IOceanPrimitive {
15:     /// @notice normalized decimals to be compatible with the Ocean.
```

### [N-06] @openzeppelin/contracts should be upgraded to a newer version

These contracts import contracts from `@openzeppelin/contracts`, but they are not using [the latest version](https://github.com/OpenZeppelin/openzeppelin-contracts/releases).

The imported version is `^4.8.1`.

<details>
<summary>There are 8 instances (click to show):</summary>

- *Curve2PoolAdapter.sol* ( [6](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L6), [6](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L6) ):

```solidity
6: import { IERC20Metadata } from "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";

6: import { IERC20Metadata } from "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";
```

- *CurveTricryptoAdapter.sol* ( [6](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L6), [6](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L6) ):

```solidity
6: import { IERC20Metadata } from "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";

6: import { IERC20Metadata } from "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";
```

- *OceanAdapter.sol* ( [6](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L6), [6](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L6) ):

```solidity
6: import { IERC20 } from "@openzeppelin/contracts/token/ERC20/IERC20.sol";

6: import { IERC20 } from "@openzeppelin/contracts/token/ERC20/IERC20.sol";
```

- *Ocean.sol* ( [9](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L9), [9](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L9) ):

```solidity
9: import { IERC20Metadata } from "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";

9: import { IERC20Metadata } from "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";
```

</details>

### [N-07] Each interfaces should be defined in a separate file

Each interface should be defined in a separate file, making it easier to import, and easier to develop and maintain. This is the strategy adopted by most popular libraries.

There is 1 instance:

- *CurveTricryptoAdapter.sol* ( [10](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L10) ):

```solidity
10: interface IWETH {
```

### [N-08] Typos

All typos should be corrected.

<details>
<summary>There are 10 instances (click to show):</summary>

- *Ocean.sol* ( [39](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L39), [49](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L49), [55](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L55), [65](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L65), [251](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L251), [275](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L275), [322](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L322), [349](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L349), [595](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L595), [647](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L647) ):

```solidity
/// @audit Suporting
39:  *  2. Suporting this defi framework is an accounting system that can transfer,

/// @audit exection
49:  *  external contracts at certain points in its exection. The lifecycle is

/// @audit recieve
55:  *   token into a liquidity pool to recieve liquidity provider tokens, the

/// @audit imlementation
65:  *  3. Read through the imlementation of Ocean.doMultipleInteractions(), again

/// @audit modifer
251:      * @dev MUST HAVE onlyApprovedForwarder modifer.

/// @audit modifer
275:      * @dev MUST HAVE onlyApprovedForwarder modifer.

/// @audit prefering
322:      * @dev We don't revert, prefering to let the external contract

/// @audit prefering
349:      * @dev We don't revert, prefering to let the external contract

/// @audit ouput
595:      * @return outputAmount The amount of ouputToken that the user is gaining

/// @audit preceeding
647:             // See the comment in the preceeding else if block.
```

</details>

### [N-09] Consider bounding input array length

The functions below take in an unbounded array, and make function calls for entries in the array. While the function will revert if it eventually runs out of gas, it may be a nicer user experience to require() that the length of the array is below some reasonable maximum, so that the user doesn't have to use up a full transaction's gas only to see that the transaction reverts.

There are 2 instances:

- *Ocean.sol* ( [463](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L463), [501](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L501) ):

```solidity
463:         for (uint256 i = 0; i < _idLength;) {

501:             for (uint256 i = 0; i < interactions.length;) {
```

### [N-10] Unused function parameters

Function parameters that are defined but not used may be logical errors and need to be checked to see if any logic is missing.
If the parameter is not really needed, it should be removed, otherwise it will repeatedly cause confusion and make code maintenance difficult.
If the parameter cannot be removed directly (for example, if it is an override function), its name should be removed and some comment can be appended.

There are 6 instances:

- *OceanAdapter.sol* ( [81-87](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L81-L87), [124](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L124) ):

```solidity
/// @audit inputToken
/// @audit outputToken
/// @audit outputAmount
/// @audit userAddress
/// @audit maximumInputAmount
81:     function computeInputAmount(
82:         uint256 inputToken,
83:         uint256 outputToken,
84:         uint256 outputAmount,
85:         address userAddress,
86:         bytes32 maximumInputAmount
87:     )

/// @audit tokenId
124:     function getTokenSupply(uint256 tokenId) external view override returns (uint256) {
```

