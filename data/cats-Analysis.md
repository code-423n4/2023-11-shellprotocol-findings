# 🐚 Shell V3 Advanced Analysis Report

![image](https://gist.github.com/assets/58657666/78ee6728-3da9-42ad-ad18-1d8c2dc8e240)


## Description of Kelp
Building on top of the foundational elements established in Shell v2, Shell v3 aims to enhance the Ocean's capabilities by making it compatible with external protocols which is achieved through the incorporation of adapter primitives, aligning with Ocean's existing design as a versatile DeFi integration that efficiently composes various primitives such as AMMs, lending pools, algorithmic stablecoins, NFT markets, and even those yet to be invented. Shell v3's objective is to ensure the Ocean's smooth interaction with external protocols, providing a more expansive ecosystem.

## Overview of the Contract Architecture:
The system consists of the following contracts (in-scope):

- Ocean Core Contract ([Ocean.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol)): Executes interactions with external contracts, updates internal accounting system incl. transferring, minting and burning. Includes an ERC-1155 ledger.
- Ocean Adapter Contract ([OceanAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol)): "Helper" type contract to assist the Ocean Core in computations/calculations and fetching IDs.
- Curve 2 Pool Adapter Contract ([Curve2PoolAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol)): Allows integration with Curve 2 Pools, enables swapping, adding & removing liquidity.
- Curve Tri-crypto Pool Adapter Contract ([CurveTricryptoAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol)): Allows integration with Curve Tri-crypto Pools, enables swapping, adding & removing liquidity as well.

## Contract Architecture Diagram:
![Architechture](https://gist.github.com/assets/58657666/c391f158-f1d4-410a-a34f-8d6741ab5f24)
## Codebase Quality Analysis:

### Ocean Core Contract:
- **Functionality**: Users provide lists of interactions, each one representing an external call to a contract, and the Ocean executes whilst updating the internal accounting system. Helper functionality for wrapping & unwrapping ERC20, ERC721 & ERC1155 tokens. Supports receiving ERC721 & ERC1155 tokens.
- **Security Concerns**:
    - **Primitives Re-entrancy**: Potential re-entrancy issues might occur with either wrong or intentionally malicious contracts being called.
    - **Forwarded Interactions**: Forwarded interactions could in edge cases prove maliciously set up and create vulnerable states for the protocol.
    - **Primitives Integration**: In case the contract interacts with malicious primitives could open up risks for harmful behavior, trust is completely put in the primitive's security.

- **Recommendations**:
    - **Checks & Limits**: Additional checks and limits can be imposed on forwarded interactions to ensure safety.
    - **External Integrations Review**: It is essential that primitive contracts are thoroughly reviewed since external calls could pose great threat.

### Ocean Adapter Contract:
- **Functionality**: Simple contract used as a "helper" to the core Ocean contract, assisting with fetching interaction ids, computing input and output token amounts and decimal conversions.
- **Security Concerns**:
    - **Input Validation**: The contract is quite short and simple with not many places for flaws. Taking that into consideration, functions lacked input validation which could in rare cases prove problematic.

- **Recommendations**:
    - **Impose Validation**: More input validation for the various functions that provide "helper" calculations.  

### Curve 2 Pool Adapter Contract & Curve Tri-crypto Pool Adapter Contract:
- **Functionality**: Allows for tokens to be wrapped & unwrapped, swapped as well as adding and removing liquidity.

## Centralization Risks:
- **Admin Access Control**: No centralization or over-powered admin functionality whatsoever. Looks good here.
- **Upgradeability**: Currently upgradeability is limited as there are is no way to implement a new implementation.
## Mechanism Review:
- **External Interactions Security Concerns**: The core functionality of the whole system in scope revolves around interacting with external contracts. This is always under the assumptions that they are safe. 
- **Whether Curve Pools are Future-Proof**: Future vulnerabilities or safety concerns in the Curve 2/Tri-crypto pools would affect the functionality of this protocol.
## Systemic Risks:
- **Primitives**: External calls to primitives are under the assumption they are safe. In case of malicious primitives being the target, or a newfound security flaw is in those primitives, it could bring varying degrees of issues to the protocol.
## Areas of Concern:
- **External Calls**: The contracts in scope are all relatively short, and the main logic is found in Ocean. The Ocean Helper and Curve adapters are unlikely to have issues arise in them. Although since the main mechanics are external calls to primitives, this would be ther main area of concern.
## Codebase Analysis:
- Well coded with adhering to good practices. Readability is well maintained and extensive comment documentation for all the mechanisms.
## Personal Recommendations:
- Additional safety precautions for all the user interractions/batch interractions and external calls.
- Extensive testing making sure risks are minimized.
- Additional logic for future upgradeability.

## Time Spent ⌛

|            |            |
|------------|------------|
| Start Date | 01.12.2023 |
| End Date   | 05.12.2023 |
| Total Days | 5 Days     |
| Hours      | ~20h       |

### Time spent:
25 hours