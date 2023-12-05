#  **Advanced Analysis Report for  <img src="https://shellprotocol.io/static/img/logo-expanded.png" alt="Shell Banner" width="200" height="auto"/>** 

[<img src="https://code4rena.com/_next/image?url=https%3A%2F%2Fstorage.googleapis.com%2Fcdn-c4-uploads-v0%2Fuploads%2FYeAbAEHLV2t.0&w=96&q=75" alt="Shell Logo" width="15" height="15"/> Overview](#1-overview)

[<img src="https://code4rena.com/_next/image?url=https%3A%2F%2Fstorage.googleapis.com%2Fcdn-c4-uploads-v0%2Fuploads%2FYeAbAEHLV2t.0&w=96&q=75" alt="Shell Logo" width="15" height="15"/> Risks to protocol](#2-risks-to-protocol)

[<img src="https://code4rena.com/_next/image?url=https%3A%2F%2Fstorage.googleapis.com%2Fcdn-c4-uploads-v0%2Fuploads%2FYeAbAEHLV2t.0&w=96&q=75" alt="Shell Logo" width="15" height="15"/> Audit approach](#3-audit-approach)

[<img src="https://code4rena.com/_next/image?url=https%3A%2F%2Fstorage.googleapis.com%2Fcdn-c4-uploads-v0%2Fuploads%2FYeAbAEHLV2t.0&w=96&q=75" alt="Shell Logo" width="15" height="15"/> Conclusions](#4-conclusions)

[<img src="https://code4rena.com/_next/image?url=https%3A%2F%2Fstorage.googleapis.com%2Fcdn-c4-uploads-v0%2Fuploads%2FYeAbAEHLV2t.0&w=96&q=75" alt="Shell Logo" width="15" height="15"/> Resources](#5-resources)


###  **1. Overview**
#### **1.1 Protocol and Architecture Overview**

- Shell Protocol, a creation Cowri Labs Inc. ais a set of smart contracts which aims to revolutionize the DeFi landscape by embracing a modular approach. The innovative architecture simplifies smart contract development and enables users to batch multiple transactions, all while minimizing gas fees. Shell Protocol's clean, modular contracts streamline development and enhance user experience, heralding a new era of accessible, efficient DeFi.
  
- The protcol comprises two major components the Ocean and the Proteus which work in tandem to deliver a powerful and adaptable DeFi ecosystem. The Ocean's shared accounting system facilitates the seamless integration of various DeFi primitives -  AMMs, lending pools, algorithmic stablecoins, NFT markets etc. while Proteus's versatile AMM engine empowers users to create highly efficient custom pools. This synergy positions Shell Protocol as a transformative force in the DeFi landscape.

- The Ocean is the main focus of this analysis. It is a single smart contract that serves as a central hub for the protocol's interactions. It simplifies the composition of DeFi primitives by providing a unified accounting system, a single ledger for all tokens types, and a standardized accounting framework. The Ocean also tracks intermediate balance updates in memory instead of storage to improve gas efficiency. This innovative approach makes Shell Protocol a versatile and efficient platform for building and using DeFi applications.

 <img src="https://github-production-user-asset-6210df.s3.amazonaws.com/112232336/287530595-1c24c85e-38b5-43c7-ab2d-de346b58ff2b.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20231203%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20231203T154418Z&X-Amz-Expires=300&X-Amz-Signature=041611ce979ff868d6ccb7236e143341b602b85923fcbdec31f1bdc172893dfe&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=0" alt="Ocean Overview" width="auto" height="auto"/>
 
#### **1.2 CodeBase Overview**

##### **1.2.1 Codebase**

- For the purpose of this audit, Shell v3 Protocol consists of four smart contracts totaling 993 SLoC. Its core design principle is composition, enabling efficient and flexible integration of various DeFi primitives. It utilizes six external calls to facilitate communication with other blockchain components. It is planned to be deployed on Arbitrum chain. The Ocean, the protocol's accounting contract, complies with the ERC1155 and ERC165 token standards. It operates independently of oracles and sidechains and represents an upgrade from its predecessor, Shell v2.

- Like other codebases built using composition patterns, the Shell protocol's code was initially challenging to understand. Nonetheless, the code is thoroughly commented, providing detailed explanations for each function's functionality although, it doesnt fully follow NatSpec. The provided documentations, the whitepapers, are top tier. They go into great details about how the protocol works, how the Ocean interacts with other primitives. It explains the reasons for the decisions made in the codebases, and overall made the codebase more understandable.

- Some basic contract safety measures also seemed to be ignored. Castings are done unsafely, there are non zero checks when making initializations, state variables are not reasonably capped, checks-effects-interaction pattern is ignored and so on. These are by no means very serious issues, but they cause a host of inconvienieces to the protocol implementation. One error on the part of the developers was using an Arbitrum incompatible solidity version, 0.8.20 - due to the PUSH0 code. Recommend switching to an earlier solidity version, preferably 0.8.10 so as to match the current V2 implementation.

- Error handling is a bit odd, with the protocol implementing the outdated `assert` errors. A mix of custom, assert and require errors is implemented. To promote uniformity across the codebase and also for best practice. we recommend updating these with custom errors preferabley, as they offer superior gas efficiency. Events are well handled, emitted for important parameter changes, although in some cases, they seemed to not follow the checks-effects-interaction patterns. This should be fixed for best practices and to protect from reentrancy. Also, to save gas, its recommended to use assembly to emit events.
  
- The protocol demonstrates great test coverage, 98%. Hardhart and Foundry tests exist for the contracts. Unit and fuzzing tests are implemented for the codebase. More tests can be conducted, to improve coverage, uncover more rudimentary bugs, strengthen modularity, and elevate overall security measures.
  
##### **1.2.2 Contract Overview**
The codebase in scope is divided into two major modules:

**Accounting contract**

- **[Ocean.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol)** - serves as the accounting backbone of the Shell Protocol, seamlessly integrating various DeFi primitives into a unified platform. It functions as a shared multi-token ledger, adhering to the ERC-1155 standard, and efficiently tracks intermediate balance updates in memory instead of storage. It streamlines token management and enables the creation of new tokens directly within the Ocean contract. Additionally, the Ocean implements a standardized accounting framework for DeFi primitives, facilitating the composition of complex DeFi transactions with enhanced gas efficiency. It comprises 561 SLoC.

**Adapter contracts**

- **[OceanAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol)** - acts as a supporting component for other adapters within the protocol. It orchestrates the necessary transformations and interactions between the Ocean, the central accounting system, and various DeFi primitives. Notably, it shouldn't hold any tokens directly. It comprises 94 SLoC.
  
- **[Curve2PoolAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol)** - facilitates integration between the protocol and the curve2pool, enabling users to perform token swaps, add liquidity, and remove liquidity for the curve usdc-usdt pool. It comprises 139 SLoC.

- **[CurveTricryptoAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol)** - bridges the protocol with the curve tricrypto pool, empowering users to execute token swaps, inject liquidity, and withdraw liquidity for the curve usdt-wbtc-eth pool. It comprises 199 SLoC.

##### **1.2.3 Contract Interactions chart**

<img src="https://github-production-user-asset-6210df.s3.amazonaws.com/112232336/287539692-9e8e0a95-2893-49df-9fd8-d29841ddba01.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20231203%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20231203T180350Z&X-Amz-Expires=300&X-Amz-Signature=79a4fee2bd52f03954030eee042236b1822cc5d6b6b1a413e4e7b45465abe702&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=0" alt="Interactions chart" width="120%" height="auto"/>

### **2. Risks to protocol**

- **Centralization** exists but not a lot, certain function are limited to the Ocean, Owner and ApprovedForwarders. It goes without saying that actions from these parties have impacts on the protocol. And they're vulnerable points of failure. One odd decision made by the team was to split the user and User `ApprovedForwarder` functions into two when they essentially do the same thing (See Oceans.sol [L210](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L210), [L256](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L256), [L229](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L229), [L281](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L281)). We think the team can find a work around for this.
    
- **ThirdParty dependencies** is also a point of entry of attack to the protocol. The contract imports OZ contracts version 4.8.1 which has a number of known issues, this should be updated, as these vulnerabilities in these contracts are likely to have an effect in the system. Also, while not in scope, the OZ Ownable was used, which is not as safe as its counterpart Ownable2Step. Recommend updating to the latter instead. The curve adapters are major points of risks to the protocol. The `CurveTricryptoAdapter` contract is the adapter for the Arbirtrum Tricrypto pool, which Curve Finance has deemed [vulnerable](https://twitter.com/CurveFinance/status/1685933800088391680) and advised users to not use. 

- **Smart Contract vulnerabilities** also pose a significant threat to the security and integrity of decentralized protocols. These vulnerabilities can manifest as critical security flaws, enabling malicious actors to exploit the protocol and steal funds or manipulate the system. Logical inconsistencies in the code can also lead to unexpected behavior and unintended consequences, potentially causing financial losses or compromising the protocol's overall functionality. Constant sytem upgrades and audits are recommended, to keep the protocol up to date with latest technologies.

- Other risks include chain reorg due to Arbitrum's optimistic rollup nature, non-standard ERC20 tokens, compromised primitives, economic attacks and so on

### **3. Audit approach**

- **Documentation review** - We reviewed the website, readme, whitepaper and explanations provided by the devs. While this was going on, we ran the contracts through static analyzer and compared the generated reports to the bot report and removed false positives.

- **Manual code review** - Here, we manually reviewed the codebase, ran provided tests, tested out various attack vectors. We looked for ways to break the protocol invariants, while making sure the contracts comforom to the needed standarfs. We also tested out the functions' logic to make sure they work as intended.

- **Codebase comparisons** - After the manual review, we studied similar protocol types, previous audits, previous commits and made comparisons to issues found in these protocols to see if the same mistakes were repeated and how effective the provided fixes were. 

- We made notes on the issues we found and prepared the needed reports.

### **4. Conclusions**
- The Shell v3 protocool aims to improve on the fundamentals developed in Shell v2, and intends to make the Ocean compatible with external protocols. It does this by introducing the OceanAdapter contracts which is a generalized adapter interface for adapter primitives.
  
- In general, the protcol seems well designed, but identified risks need to be fixed. Measures should be implemented to protect the protocol from potential attacks. We recommend cleaning up the codebase and mititgating any identified vulnerabilities before deployment. This proactive approach will ensure the protocol's continued robustness and securit

### **5. Resources**
- [Current Audit Page](https://code4rena.com/contests/2023-11-shell-protocol#top)

- [Website](https://shellprotocol.io/)

- [Documentation](https://wiki.shellprotocol.io/how-shell-works/the-ocean-accounting-hub)

- [Ocean White Paper](https://shellprotocol.io/static/Ocean_-_Shell_v2_Part_2.pdf)

- [Previous C4 Audit Report](https://code4rena.com/reports/2023-08-shell)




### Time spent:
30 hours