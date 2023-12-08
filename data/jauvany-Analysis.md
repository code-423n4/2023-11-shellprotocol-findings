**Overview**

Shell v3 improves upon the fundamentals developed in Shell v2. The goal of Shell v3 is to make the Ocean compatible with external protocols through the use of adapter primitives.

**Scope**

> Ocean.sol
This contract is the main contract of the protocol. The accounting engine of the shell protocol. It uses oppenzepplin libraries.

> Curve2PoolAdapter.sol
This contract is the adapter that enables integration with the curve 2 pool. It uses oppenzepplin libraries.

> CurveTricryptoAdapter.sol
This contract is the adapter that enables integration with the curve tricrypto pool.  It uses oppenzepplin libraries.

> OceanAdapter.sol
This ontract is the Helper contract for the above mentioned adapters. Just like the other contracts,  it uses oppenzepplin libraries.

# Approach taken in evaluating the codebase
 
**Time Spent**
> A total of 3 days (24 hours) were dedicated to completing this analysis, distributed as follows: 
– Day 1: I spent time reading the different available documentation in order to have a deep understanding of the protocol.
 
- Day 2: I analyzed the codebase for better  understanding, Performed a Mechanism review and investigated possible systemic risks, and centralization risks. 
 
- Day 3: I dedicated this day to coming up with possible Architecture recommendations and Lastly, I prepared the final analysis report.

# Codebase quality analysis

**Ocean.sol**
- The Contract is non-functional due to incompatible Solidity version with Arbitrum.
- The Contract is vulnerable to centralization risk.
- User facing functions like ```doInteraction() and doMultipleInteractions()``` should have address(0) checks.
- The contract uses assert instead of require.
- The ```changeUnwrapFee()``` critical function lacks two-step update.
- The contract contains the ```decimals()``` function which is not part of the ERC20 standard.
- Some tokens may revert when zero value transfers are made
- Contract lacks maximum value checks to ensure that state variables cannot be set to values that may excessively harm users.
 
**Curve2PoolAdapter.sol**
- The Contract is non-functional due to incompatible Solidity version with Arbitrum.
- The contract contains the ```decimals()``` function which is not part of the ERC20 standard.
- The contract does not consider some tokens of type(uint256).max as an infinite approval.
- The Contract uses infinite approvals with no means to revoke.
- Contract lacks maximum value checks to ensure that state variables cannot be set to values that may excessively harm users.
 
**CurveTricryptoAdapter.sol**
- The Contract is non-functional due to incompatible Solidity version with Arbitrum.
- The contract contains the ```decimals()``` function which is not part of the ERC20 standard.
- The contract does not consider some tokens of type(uint256).max as an infinite approval.
- The Contract uses infinite approvals with no means to revoke.
- Contract lacks maximum value checks to ensure that state variables cannot be set to values that may excessively harm users.
 
**OceanAdapter.sol**
- The Contract is non-functional due to incompatible Solidity version with Arbitrum.
- The Contract is vulnerable to centralization risk.
- Contract lacks maximum value checks to ensure that state variables cannot be set to values that may excessively harm users.

 
# Mechanism review

- The mechanisms implemented in Shell Protocol, including the ERC-1155 standard, the ERC-165 standards, and the various modules for ownership management, execution and extension, are well-designed. They provide a comprehensive range of features and capabilities, enabling a wide range of use cases.

- Non the less, there are some potential issues and risks associated with these mechanisms. For example, the use of the ```onlyowner``` modifier exposes the protocol to centralisation risk and the use of compiler version 0.8.20 on all contracts for Shell v3 makes the contracts non-functional due to incompatible Solidity version with Arbitrum. These issues should be carefully considered and mitigated to ensure the security and reliability of the ecosystem.
 
# Centralization risks
 
- A single point of failure is not acceptable for this project. Centrality risk is high in the project as the role of onlyOwner detailed below has very critical and important powers: Project and funds may be compromised by a malicious or stolen private key onlyOwner msg.sender
 
> File: src/ocean/Ocean.sol

```
196: function changeUnwrapFee(uint256 nextUnwrapFeeDivisor) external override onlyOwner { 
```

```
256: function forwardedDoInteraction( 
257: Interaction calldata interaction, 
258: address userAddress 
259: ) 
260: external 
261: payable 
262: override 
263: onlyApprovedForwarder(userAddress) 
264: returns (uint256 burnId, uint256 burnAmount, uint256 mintId, uint256 mintAmount) 
265: {
```

```
281: function forwardedDoMultipleInteractions( 
282: Interaction[] calldata interactions, 
283: uint256[] calldata ids, 
284: address userAddress 
285: ) 
286: external 
287: payable 
288: override 
289: onlyApprovedForwarder(userAddress) 
290: returns ( 
291: uint256[] memory burnIds, 
292: uint256[] memory burnAmounts, 
293: uint256[] memory mintIds, 
294: uint256[] memory mintAmounts 
295: ) 
296: { 
```
> File: src/adapters/OceanAdapter.sol 

```
55: function computeOutputAmount( 
56: uint256 inputToken, 
57: uint256 outputToken, 
58: uint256 inputAmount, 
59: address, 
60: bytes32 metadata 
61: ) 
62: external 
63: override 
64: onlyOcean 
65: returns (uint256 outputAmount) 
66: {
```

```
81: function computeInputAmount( 
82: uint256 inputToken, 
83: uint256 outputToken, 
84: uint256 outputAmount, 
85: address userAddress, 
86: bytes32 maximumInputAmount 
87: ) 
88: external 
89: override 
90: onlyOcean 
91: returns (uint256 inputAmount) 
92: {
```

# Systemic risks

 External Contract Dependencies: Shell protocol relies on Openzepplin contracts. If any of these contracts have vulnerabilities, it would affect this contract.

 Test Coverage: The test coverage provided by Shell protocol is 98%, however I recommend 100% of the test coverage.

 Like any smart contract-based system, Shell Protocol is exposed to potential coding bugs or vulnerabilities. Exploiting these issues could result in the loss of funds or manipulation of the protocol.


# Architecture recommendations
 
 > Shell protocol’s architecture seems solid in general, none the less Here are some areas that could be improved:

- Testing and Simulations: Even though the Shell Protocol implements several tests, consider adding more tests to achieve 100% test coverage. 
Secondly: I recommend creating a live testnet app. Here is an [example](https://app.opendollar.com/) from The Open Dollar protocol. Conduct thorough testing of all contracts and functions and simulations to understand how they will behave under various market conditions.

- Compiler Version: The Shell Protocol V2 is currently live on Arbitrum mainnet (see https://app.shellprotocol.io/). This audit is for V3 and uses version 0.8.20 of Solidity whereas V2 used version 0.8.10 (see [Shell github](https://github.com/Shell-Protocol/Shell-Protocol/blob/main/src/ocean/Ocean.sol)). The compiler for Solidity 0.8.20 switches the default target EVM version to [Shanghai](https://blog.soliditylang.org/2023/05/10/solidity-0.8.20-release-announcement/#important-note), which includes the new PUSH0 op code. This op code may not yet be implemented on all L2s, so deployment on these chains will
fail. See this relevant [issue](https://github.com/ethereum/solidity/issues/14254) on the official Solidity github for reference. As the contract(s) are intended to be deployed on Arbitrum, this will cause them to be completely non-functional. To work around this issue, use an earlier EVM version, such as 0.8.19.

- Improving gas efficiency: Multiple mappings should be combined with the same key type where appropriate. 

**Other recommendations**
 
- Regular code reviews and adherence to best practices.
- Conduct external audits by security experts.
- Consider open sourcing the contract for community review.
- Maintain comprehensive security documentation.
- Establish a responsible disclosure policy for vulnerabilities.
- Implement continuous monitoring for unusual activity.
- Educate users about risks and best practices.


### Time spent:
24 hours