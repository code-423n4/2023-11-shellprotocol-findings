|     |     |     |
| --- | --- | --- |
| ID  | Description | Severity |
| 1   | Use a more recent version of OpenZeppelin dependencies | Low |
| 2   | Empty receive()/payable fallback() function does not authenticate requests | Low |
| 3   | Function Calls in Loop Could Lead to Denial of Service | Low |
| 4   | Insufficient coverage | Low |
| 5   | The `onERC721Received` callback can be used to create fake liens | Low |
| 6   | Constructors contains no validation | Low |
| 7   | Use a more recent version of Solidity | Non-Crtical |
| 8   | Use named parameters for mapping type declarations | Non-Critical |
| 9   | NatSpec is missing | Non-Critical |
| 10  | Take advantage of Custom Error’s return value property | Non-Critical |
| 11  | Use SMTChecker | Non-Critical |
| 12  | Use underscores for number literals | Non-Critical |
| 13  | tates/flags should use `Enum`s rather than separate constants | Non-Critical |
| 14  | Event is missing `indexed` fields | Non-Critical |

## \[L-01\] Use a more recent version of OpenZeppelin dependencies

**Context:** All contracts

**Description:** For security, it is best practice to use the latest OZ version.

```
package.json:
- 38:     "@openzeppelin/contracts": "^4.8.1",
+ 38:     "@openzeppelin/contracts": "^4.9.0",
```

**Recommendation:** Old version of OZ is used `(4.8.1)`, newer version can be used `(4.9.0)`

<ins>https://github.com/OpenZeppelin/openzeppelin-contracts/releases/tag/v4.9.0</ins>

&nbsp;

## \[L-02\] Empty receive()/payable fallback() function does not authenticate requests

If the intention is for the Ether to be used, the function should call another function, otherwise it should revert (e.g. require(msg.sender == address(weth))). Having no access control on the function means that someone may send Ether to the contract, and have no way to get anything back out, which is a loss of funds. If the concern is having to spend a small amount of gas to check the sender against an immutable address, the code should at least have a function to rescue unused Ether.

&nbsp;

```Solidity
File: src/adapters/CurveTricryptoAdapter.sol

291: fallback() external payable { }
```

&nbsp;

## \[L-03\] Function Calls in Loop Could Lead to Denial of Service

Function calls made in unbounded loop are error-prone with potential resource exhaustion as it can trap the contract due to the gas limitations or failed transactions. Here are some of the instances entailed:

```solidity
File: src/ocean/Ocean.sol

463: for (uint256 i = 0; i < _idLength;) {

501: for (uint256 i = 0; i < interactions.length;) {
```

&nbsp;

## \[L-04\] Insufficient coverage

Description  
The test coverage rate of the project is ~98%. Testing all functions is best practice in terms of security criteria.

Due to its capacity, test coverage is expected to be 100%.

## \[L-05\] The `onERC721Received` callback can be used to create fake liens

### Since the `onERC721Received` callback is a public function, it can be called by anyone to create fake liens. The `from` parameter is user supplied to the function, which means that anyone can create fake liens on behalf of an arbitrary lender.

```
15: import { IERC721Receiver } from "@openzeppelin/contracts/token/ERC721/IERC721Receiver.sol";
```

&nbsp;

## \[L-06\] Constructors contains no validation

In Solidity, when values are being assigned in constructors to unsigned or integer variables, it's crucial to ensure the provided values adhere to the protocol's specific operational boundaries as laid out in the project specifications and documentation. If the constructors lack appropriate validation checks, there's a risk of setting state variables with values that could cause unexpected and potentially detrimental behavior within the contract's operations, violating the intended logic of the protocol. This can compromise the contract's security and impact the maintainability and reliability of the system. In order to avoid such issues, it is recommended to incorporate rigorous validation checks in constructors. These checks should align with the project's defined rules and constraints, making use of Solidity's built-in require function to enforce these conditions. If the validation checks fail, the require function will cause the transaction to revert, ensuring the integrity and adherence to the protocol's expected behavior.

&nbsp;

&nbsp;

## \[N-02\] Use a more recent version of Solidity

For security, it is best practice to use the latest Solidity version.

For the security fix list in the versions: https://github.com/ethereum/solidity/blob/develop/Changelog.md

## \[N-03\] Use named parameters for mapping type declarations

Consider using named parameters in mappings (e.g. `mapping(address account => uint256 balance)`) to improve readability. This feature is present since Solidity 0.8.18.

```
73: mapping(uint256 => uint256) indexOf;

    /// @notice The underlying token decimals wrt to the Ocean ID
76: mapping(uint256 => uint8) decimals;
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L73

```
65: mapping(uint256 => int128) indexOf;

    /// @notice The underlying token decimals wrt to the Ocean ID
68:  mapping(uint256 => uint8) decimals;
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L65C5-L68C40

### \[N-04\] NatSpec is missing

NatSpec is missing for the following functions

```
161:     function primitiveOutputAmount(
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L161

&nbsp;

## \[N-05\] Take advantage of Custom Error’s return value property

An important feature of Custom Error is that values such as address, tokenID, msg.value can be written inside the `()` sign, this kind of approach provides a serious advantage in debugging and examining the revert details of dapps such as tenderly.

```
error INVALID_COMPUTE_TYPE();
error SLIPPAGE_LIMIT_EXCEEDED();
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L29

## \[N-06\] Use SMTChecker

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

&nbsp;

## \[N-07\] Use underscores for number literals

```
uint256 constant MIN_UNWRAP_FEE_DIVISOR = 2000; // 2_000
```

&nbsp;https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L95

&nbsp;

## \[N-08\] States/flags should use `Enum`s rather than separate constants

&nbsp;

```
uint256 constant NOT_INTERACTION = 1;
    uint256 constant INTERACTION = 2;
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L106

&nbsp;

## \[N‑09\] Event is missing `indexed` fields

Index event fields make the field more quickly accessible <ins>to off-chain tools</ins> that parse events. However, note that each index field costs extra gas during emission, so it’s not necessarily best to index the maximum allowed per event (three fields). Each `event` should use three `indexed` fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L35C4-L58C7

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L111