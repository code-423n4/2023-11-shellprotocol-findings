# 🛠️ Analysis - Shell Protocol Audit
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Analysis of the code base | What is unique? How are the existing patterns used? "Solidity-metrics" was used  |
|c) |Test analysis | Test scope of the project and quality of tests |
|d) |Security Approach of the Project | Audit approach of the Project |
|e) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|f) |Packages and Dependencies Analysis | Details about the project Packages |
|g) |Other recommendations | What is unique? How are the existing patterns used? |
|h) |New insights and learning from this audit | Things learned from the project |



## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2023-11-shellprotocol

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2023-11-shellprotocol?tab=readme-ov-file#installation)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review| [Shell](https://wiki.shellprotocol.io/how-shell-works/the-ocean-accounting-hub) |Provides a basic architectural teaching for General Architecture|
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither Report](https://github.com/Shell-Protocol/Shell-Protocol?tab=readme-ov-file#static-analysis)| |
|5|Test Suits|[Tests](https://github.com/Shell-Protocol/Shell-Protocol?tab=readme-ov-file#testing-1)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2023-11-shellprotocol?tab=readme-ov-file#scope)||
|7|Infographic|[Figma](https://www.figma.com/)|I made Visual drawings to understand the hard-to-understand mechanisms|
|8|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2023-11-shellprotocol?tab=readme-ov-file#additional-context)||

## b) Analysis of the code base

The most important summary in analyzing the code base is the stacking of codes to be analyzed.
In this way, many predictions can be made, including the difficulty levels of the contracts, which one is more important for the auditor, the features they contain that are important for security (payable functions, uses assembly, etc.), the audit cost of the project, and the time to be allocated to the audit;
Uses Consensys Solidity Metrics
-  **Lines:** total lines of the source unit
-  **nLines:** normalized lines of the source unit (e.g. normalizes functions spanning multiple lines)
-  **nSLOC:** normalized source lines of code (only source-code lines; no comments, no blank lines)
-  **Comment Lines:** lines containing single or block comments
-  **Complexity Score:** a custom complexity score derived from code statements that are known to introduce code complexity (branches, loops, calls, external interfaces, ...)


</br>
</br>

<img width="965" alt="image" src="https://github.com/code-423n4/2023-11-shellprotocol/assets/104318932/58f06035-2382-4448-8daf-3823493e35f7">

</br>
</br>

## <h1 align="center">  Ocean.sol   </h1>



### Ocean.sol -  [ Imports and Contract Summary]

<img width="1441" alt="image" src="https://github.com/code-423n4/2023-11-shellprotocol/assets/104318932/ca58810a-2eb2-4496-94ee-d31dfc2c47db">

### Ocean.sol - [ State Variables - Mappings - Modifiers - Events]

<img width="1167" alt="image" src="https://github.com/code-423n4/2023-11-shellprotocol/assets/104318932/e93b2468-56a7-4f0e-8d86-51160b780ae9">


</br>
</br>

### Ocean.sol - [ doMultipleInteractions and _doMultipleInteractions functions]

<img width="1749" alt="image" src="https://github.com/code-423n4/2023-11-shellprotocol/assets/104318932/04c0354f-2a54-43bc-9aee-bddf84bdb44a">

</br>
</br>

### Ocean.sol - [ doInteraction and _doInteraction functions]

<img width="1116" alt="image" src="https://github.com/code-423n4/2023-11-shellprotocol/assets/104318932/67aec51c-ac90-4f76-9fed-14276195dce8">

</br>
</br>


## c) Test analysis
### What did the project do differently? ;
-   1) It can be said that the developers of the project did a quality job, there is a test structure consisting of tests with quality content.




### What could they have done better?


-  1) In order to understand the test scenarios and develop more effective test scenarios, the following bob, alice and other roles are can be defined one by one, in this way role definitions increase the quality and readability in tests

```solidity

 // Sample labels
vm.label(bob, 'bob');
vm.label(alice, 'alice');
vm.label(DEPLOYER, 'deployer');
vm.label(USDE_OWNER, 'usde owner');
vm.label(POOL_PROXY, 'lending pool');
```
</br>

-  2) Test suites do not test for re-entrancy Test teams are testing many functions and variables, but recently, due to the vulnerability in the Vyper Compiler, the hacking of the projects using certain Vyper compiler and losing 50 million $ has revealed the security weakness here. 
https://cointelegraph.com/news/curve-finance-pools-exploited-over-24-reentrancy-vulnerability
The accuracy of the functions has been tested, but it has not been tested whether the nonReentrant modifier in the function works correctly or not. For this, a test must be written that tries to reentrancy and was observed to fail.

</br>

-  3) If we look at the test scope and content of the project with a systematic checklist, we can see which parts are good and which areas have room for improvement As a result of my analysis, those marked in green are the ones that the project has fully achieved. The remaining areas are the development areas of the project in terms of testing ;


<img width="783" alt="image" src="https://github.com/code-423n4/2023-11-shellprotocol/assets/104318932/d6cd6613-c38b-436e-b812-d547327642ee">
Ref:https://xin-xia.github.io/publication/icse194.pdf

</br>
</br>

## d) Security Approach of the Project

### Successful current security understanding of the project;

1 - First they did the main audit from Consensys Diligence and Trail of Bits and resolved all the security concerns in the report 

2- They manage the 2nd audit process with an innovative audit such as Code4rena, in which many auditors examine the codes.


### What the project should add in the understanding of Security;

1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)

2- After the project is published on the mainnet, there should be emergency action plans (not found in the documents)

3- Emergency Action Plan
In a high-level security approach, there should be a crisis handbook like the one below and the strategic members of the project should be trained on this subject and drills should be carried out. Naturally, this information and road plan will not be available to the public.
https://docs.google.com/document/u/0/d/1DaAiuGFkMEMMiIuvqhePL5aDFGHJ9Ya6D04rdaldqC0/mobilebasic#h.27dmpkyp2k1z

4- ChainAnalysis oracle
With the ChainAnalysis oracle, OFAC interaction can be blocked so that legal issues do not arise

</br>
</br>

## e) Other Audit Reports and Automated Findings 

**Automated Findings:**
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/bot-report.md


**Other Audit Reports (Consensys Diligence and Trail of Bits):**
https://wiki.shellprotocol.io/getting-started/security-and-bounties#audits


</br>
</br>

##  f) Packages and Dependencies Analysis 📦

| Package                                                                                                                                     | Version                                                                                                               | Usage in the project                               | Audit Recommendation                                                                                                             |
| ------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| [`openzeppelin`](https://www.npmjs.com/package/@openzeppelin/contracts)         | [![npm](https://img.shields.io/npm/v/@openzeppelin/contracts.svg)](https://www.npmjs.com/package/@openzeppelin/contracts)     |               @openzeppelin contracts  |-  Version `4.8.1` is used by the project, it is recommended to use the newest version `5.0.0` |


</br>
</br>




## g) Other recommendations

✅ The use of assembly in project codes is very low, I especially recommend using such useful and gas-optimized code patterns; https://github.com/dragonfly-xyz/useful-solidity-patterns/tree/main/patterns/assembly-tricks-1

✅ A good model can be used to systematically assess the risk of the project, for example this modeling is recommended; https://www.notion.so/Smart-Contract-Risk-Assessment-3b067bc099ce4c31a35ef28b011b92ff#7770b3b385444779bf11e677f16e101e

✅ All staff accounts for the project should have control policies that require 2FA and must use 2FA wherever possible.
100% comprehensive security cannot be achieved based on smart contract codes alone.
Implement a more comprehensive policy to enforce non-SMS 2FA.
You can find the latest Simswap attack on Code4rena and details about it in this article: 
https://medium.com/code4rena/code4rena-twitter-x-incident-8b7f308a555d

✅ The Reentrancy modifier used by Opensea is more gas optimized and battle tested, I recommend you replace the reentrancy structure in the project with this code;
https://github.com/ProjectOpenSea/seaport/blob/main/contracts/lib/ReentrancyGuard.sol

✅ I recommend you to set up a `system.sol` basic architecture where all contracts are registered.
The entire system can revolve around a single contract, like SystemRegistry. This is the contract that ties all the other contracts together, and from this contract we should be able to list all the other contracts in the system. It's almost like a registry. 

</br>
</br>

## h) New insights and learning from this audit 


🔎 1- Updates to The Ocean in Shell v3:
Significant changes include the removal of reentrancy guards for specific methods, enabling Ether wrapping, and a refactoring of the order in which a primitive's balances are updated. These updates facilitate better interaction with external protocols and improve the efficiency of token handling.

🔎 2- Liquidity Pools Adaptation:
The LiquidityPoolProxy.sol has been refactored to align with the updated balance management in The Ocean. This involves adjusting values post-balance retrieval, ensuring seamless integration with the new system.

🔎 3- Introduction of Adapter Primitives:
Shell v3 introduces OceanAdapter.sol, a generalized interface for adapter primitives, along with example implementations like Curve2PoolAdapter.sol and CurveTricryptoAdapter.sol. These serve as templates for integrating various external protocols with The Ocean.

🔎 4- Invariants and Security Measures in Shell v3:
The project outlines several invariants to ensure security and consistency, such as user balance protection, adherence to ERC standards, and specific conditions under which the Ocean can make external transfers or mint tokens. It also notes the Ocean's non-support for rebasing tokens and fee on transfer tokens, along with the implementation of reentrancy checks for enhanced security.

### Time spent:
16 hours