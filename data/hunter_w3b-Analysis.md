# Analysis - Shell Protocol Contest

![Shell Protocol](https://code4rena.com/_next/image?url=https%3A%2F%2Fstorage.googleapis.com%2Fcdn-c4-uploads-v0%2Fuploads%2FYeAbAEHLV2t.0&w=256&q=75)

## Description overview of The Shell Protocol Contest

![Shell](https://497184007-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FHcJAAgnAk36Ru4lEnsmj%2Fuploads%2FbiaovJ8uK4Ai3NuBx79N%2Fimage.png?alt=media&token=653a1a6d-3d5a-47fa-9c59-dd8173dd6af3)
Shell Protocol, built on Arbitrum One, is a modular DeFi ecosystem that simplifies interactions for users and developers
by offering one-click transactions, efficient Automated Market Makers (AMMs), and a clean, modular architecture, with
Shell v3 enhancing compatibility with external protocols through adapter primitives.

**The key contracts of Shell protocol for this Audit are**:

The key contracts of Shell Protocol for this audit include Ocean.sol, the accounting engine of the protocol,
Curve2PoolAdapter.sol, an adapter facilitating integration with the Curve 2 pool, and CurveTricryptoAdapter.sol, an
adapter enabling integration with the Curve Tricrypto pool. Additionally, OceanAdapter.sol serves as a helper contract
for these adapters, providing a generalized interface.

- **Ocean.sol**: Serving as a multitoken ledger, Ocean interacts with contracts implementing IERC20, IERC721, IERC-1155,
  or IOceanPrimitive, functioning as a comprehensive accounting system for transfers, mints, or burns of tokens with
  unique oceanIds. It also supports an ERC-1155 ledger for user and primitive interactions, managing token wrapping and
  unwrapping, and executing fee calculations.

- **Curve2PoolAdapter.sol**: This OceanAdapter facilitates swapping, adding liquidity, and removing liquidity for the
  Curve USDC-USDT pool, utilizing Ocean interactions. It specifically handles token wrapping and unwrapping, with a
  focus on supporting the Curve 2pool's functionalities.

- **CurveTricryptoAdapter.sol**: Tailored for the Curve USDT-WBTC-ETH pool, this OceanAdapter enables swapping, adding
  liquidity, and removing liquidity. It includes functionalities such as token wrapping, unwrapping, and slippage
  protection.

- **OceanAdapter.sol**: As an abstract helper contract for Shell adapters, OceanAdapter provides common functionality
  for token wrapping, unwrapping, and handling interactions with external primitives within the Ocean ecosystem.

As the audit of these contracts, I checked for potential security vulnerabilities, such as reentrancy, access control
issues, and logic flaws. Additionally, thoroughly test the functions and roles defined in these contracts to make sure
they behave as expected.

## System Overview

### Scope

- **Ocean.sol**: The Ocean contract, serving as a public multitoken ledger for DeFi, facilitates token interactions with
  external contracts, including wrapping and unwrapping ERC-20, ERC-721, and ERC-1155 tokens, while maintaining a
  comprehensive accounting system; it also incorporates an efficient fee system for unwrapping based on the specified
  fee divisor and supports both direct and forwarded interactions executed on behalf of users by approved forwarders.

  ![Ocean.sol](https://i.im.ge/2023/12/09/EsSpcG.Ocean.png)

- **Curve2PoolAdapter.sol**: Curve2PoolAdapter.sol contract is an adapter for the Curve Finance 2pool, designed to
  facilitate swapping, adding liquidity, and removing liquidity for the Curve USDC-USDT pool; it includes functions for
  wrapping and unwrapping tokens, as well as handling various interactions with the Ocean protocol; the contract manages
  Ocean IDs for x, y, and lp tokens, mapping them to corresponding Curve pool indices, and utilizes mappings for token
  decimals with respect to the Ocean ID; the adapter is constructed to initialize immutables, mappings, and approve
  tokens, providing a bridge between the Ocean protocol and the Curve 2pool.

  ![Curve2PoolAdapter.sol](https://user-images.githubusercontent.com/70327041/289010428-18bba768-ca85-44ed-abd8-f55c66e98898.png)

- **CurveTricryptoAdapter.sol**: serves as an adapter for the Curve Finance Tricrypto pool, enabling functionalities
  such as swapping, adding liquidity, and removing liquidity for the Curve USDT-WBTC-ETH pool; it establishes mappings
  for token Ocean IDs, corresponding Curve pool indices, and token decimals; the adapter is initialized in the
  constructor to manage the immutables, mappings, and token approvals, providing a seamless connection between the Ocean
  protocol and the Curve Tricrypto pool; the contract also includes functions for wrapping and unwrapping tokens, and it
  implements logic for handling various interactions, emitting events, and enforcing slippage limits.

  ![CurveTricryptoAdapter.sol](https://i.im.ge/2023/12/09/EsSnDa.CurveTricryptoAdapter.png)

- **OceanAdapter**: This contract acts as a helper contract for shell adapters in the context of the Ocean protocol,
  providing functionality for wrapping and unwrapping tokens and enabling seamless interaction with external primitives;
  it abstractly implements the IOceanPrimitive interface, defining methods like computeOutputAmount, computeInputAmount,
  and others, allowing the Ocean to manage input and output tokens for various interactions; the contract initializes
  immutable parameters such as Ocean and external primitive addresses, and it includes utility functions for calculating
  Ocean IDs, fetching interaction IDs, and converting decimals; it enforces onlyOcean modifier to restrict method calls
  to the Ocean protocol and implements ERC1155-related functions for token handling.

  ![OceanAdapter.sol](https://i.im.ge/2023/12/09/EsdM6z.OceanAdapter.png)

### Privileged Roles

Some privileged roles exercise powers over the controller of the contracts:

- **Ocean**

```solidity
    modifier onlyOcean() {
        require(msg.sender == ocean);
        _;
    }
```

- **Approved-Forwarder**

```solidity
    modifier onlyApprovedForwarder(address userAddress) {
        if (!isApprovedForAll(userAddress, msg.sender)) revert FORWARDER_NOT_APPROVED();
        _;
    }
```

## Approach Taken-in Evaluating The Shell Protocol

Accordingly, I analyzed and audited the subject in the following steps;

1.  **Core Protocol Contract Overview**:

    I focused on thoroughly understanding the codebase and providing recommendations to improve its functionality. The
    main goal was to take a close look at the important contracts and how they work together in the Shell Protocol. I
    started with the Ocean contract that is at the core of the system and from which other contracts interact or depend
    on. In this case.

    I start with the following contracts, which play crucial roles in the Shell v3 protocol system:

    **Main Contracts I Looked At**

                Ocean.sol
                Curve2PoolAdapter.sol
                CurveTricryptoAdapter.sol
                OceanAdapter.sol

    I started my analysis by examining the Ocean.sol contract, which serves as the main contract for the Shell v3
    Protocol. This contract acting as a public multitoken ledger designed to interact seamlessly with diverse token
    standards, including IERC20, IERC721, IERC-1155, and IOceanPrimitive.Its core features include an intricate
    accounting system for token management, support for ERC-1155 with standard functionality, and a dynamic unwrap fee
    mechanism. Additionally, the contract provides functionality for wrapping and unwrapping ERC20, ERC721, and ERC1155
    tokens.

    Then, I turned my attention to the Curve2PoolAdapter.sol because it appears to be a crucial component for
    facilitating swapping, adding liquidity, and removing liquidity in the Curve USDC-USDT pool. The contract's core
    functions involve wrapping and unwrapping tokens in the Ocean, determining ComputeType for different interactions,
    and executing swaps, deposits, or withdrawals in the Curve pool based on specified parameters.

    Then dive into the CurveTricryptoAdapter.sol contract because it serves as the Ocean Protocol adapter for
    interacting with the Curve Tricrypto pool, enabling operations such as swapping, adding liquidity, and removing
    liquidity for the pool containing USDT, WBTC, and ETH.

    Then audit the OceanAdapter.sol contract, a helper contract for shell adapters in the Shell Protocol. It defines the
    normalized decimals constant and maintains Ocean and primitive addresses, along with a mapping of underlying tokens.

2.  **Documentation Review**:

    Then went to Review
    [This Docs](https://docs.google.com/document/d/1HJpomEsY4dAyXsQ3MF27YFaX74QS4EE05f76ikLg25c/edit) for a more
    detailed and technical explanation of Shell protocol and then learn more about v3 improves in
    [these](https://wiki.shellprotocol.io/how-shell-works/the-ocean-accounting-hub)
    [res](https://github.com/Shell-Protocol/Shell-Protocol#the-ocean).

3.  **Compiling code and running provided tests**:

4.  **Manuel Code Review** In this phase, I initially conducted a line-by-line analysis, following that, I engaged in a
    comparison mode.

    - **Line by Line Analysis**: Pay close attention to the contract's intended functionality and compare it with its
      actual behavior on a line-by-line basis.

    - **Comparison Mode**: Compare the implementation of each function with established standards or existing
      implementations, focusing on the function names to identify any deviations.

## Architecture Description and Diagram

Architecture of the contracts that are part of the Shell protocol:

![Diagram](https://i.im.ge/2023/12/09/EsdQV6.Screenshot-from-2023-12-08-12-32-16.png)

### Ocean.sol:

- Purpose: Ocean.sol is the central contract serving as a public multitoken ledger for DeFi.

- Key Functionalities: Handles interactions with external contracts, facilitating wrapping and unwrapping of ERC-20,
  ERC-721, and ERC-1155 tokens. Maintains a comprehensive accounting system to track various token types. Implements an
  efficient fee system for unwrapping based on a specified fee divisor. Supports direct and forwarded interactions
  executed on behalf of users by approved forwarders.

### Curve2PoolAdapter.sol:

- Purpose: Curve2PoolAdapter.sol acts as an adapter for the Curve Finance 2pool, specifically designed for the Curve
  USDC-USDT pool.

- Key Functionalities: Facilitates swapping, adding liquidity, and removing liquidity within the Curve pool. Manages
  Ocean IDs for x, y, and lp tokens, mapping them to corresponding Curve pool indices. Utilizes mappings for token
  decimals concerning the Ocean ID. Initializes immutables, mappings, and approves tokens to act as a bridge between the
  Ocean protocol and the Curve 2pool.

### CurveTricryptoAdapter.sol:

- Purpose: CurveTricryptoAdapter.sol serves as an adapter for the Curve Finance Tricrypto pool, specifically for the
  Curve USDT-WBTC-ETH pool.

- Key Functionalities: Enables swapping, adding liquidity, and removing liquidity within the Curve Tricrypto pool.
  Establishes mappings for token Ocean IDs, corresponding Curve pool indices, and token decimals. Manages immutables,
  mappings, and token approvals, providing a seamless connection between the Ocean protocol and the Curve Tricrypto
  pool. Includes functions for wrapping and unwrapping tokens, implementing logic for various interactions, emitting
  events, and enforcing slippage limits.

### OceanAdapter:

- Purpose: OceanAdapter acts as a helper contract for shell adapters within the Ocean protocol.

- Key Functionalities: Provides functionality for wrapping and unwrapping tokens. Implements the IOceanPrimitive
  interface, defining methods like computeOutputAmount and computeInputAmount. Abstracts common functionalities for
  shell adapters, allowing for a consistent interface. Initializes immutable parameters such as Ocean and external
  primitive addresses. Includes utility functions for calculating Ocean IDs, fetching interaction IDs, and converting
  decimals. Enforces the onlyOcean modifier to restrict method calls to the Ocean protocol. Implements ERC1155-related
  functions for token handling.

![Diagram-1](https://i.im.ge/2023/12/09/EsSRCm.arch1.png)

### Some potential areas for improvement and Architecture feedback

1. Consider upgradeability design if future changes are expected via upgrade proxies like EIP-1167.

2. Introduce formal verification of key invariants, constraints and upgrade processes to prevent unintended behaviors.

3. In OceanAdapter contract the onERC1155Received function is currently implemented to return a fixed selector.
   Depending on the use case, consider implementing a more meaningful logic to handle ERC1155 tokens.

4. Enhance inline comments to provide more detailed explanations, especially for complex logic or business rules. This
   will make it easier for developers and auditors to understand the purpose and functionality of each part of the code.

5. Introduce a multi-sig wallet for the DAO funds and restrictions on minting/issuance to prevent misuse if any single
   signer is compromised.

6. Continuously audit code with expert security reviewers and bug bounties to proactively identify issues.

## Codebase Quality

Overall, I consider the quality of the Shell protocol codebase to be Excellent. The code appears to be mature and
well-developed. We have noticed the implementation of various standards adhere to appropriately. Details are explained
below:

| Codebase Quality Categories              | Comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Code Maintainability and Reliability** | The Shell Protocol contracts demonstrates good maintainability through modular structure, consistent naming, and efficient use of helper contract. It also prioritizes reliability by handling errors, conducting security checks. However, for enhanced reliability, consider professional security audits and documentation improvements. Regular updates are crucial in the evolving space.                                                                                                                                                                                                                                                                                      |
| **Code Comments**                        | During the audit of the Shell contracts codebase, I found that some areas lacked sufficient internal documentation to enable independent comprehension. While comments provided high-level context for certain constructs, lower-level logic flows and variables were not fully explained within the code itself. Following best practices like NatSpec could strengthen self-documentation. As an auditor without additional context, it was challenging to analyze code sections without external reference docs. To further understand implementation intent for those complex parts, referencing supplemental documentation was necessary.                                      |
| **Documentation**                        | The documentation of the Shell project is quite comprehensive and detailed, providing a solid overview of how Shell v3 protocol is structured and how its various aspects function. However, we have noticed that there is room for additional details, such as diagrams, to gain a deeper understanding of how different contracts interact and the functions they implement. With considerable enthusiasm. We are confident that these diagrams will bring significant value to the protocol as they can be seamlessly integrated into the existing documentation, enriching it and providing a more comprehensive and detailed understanding for users, developers and auditors. |
| **Testing**                              | The audit scope of the contracts to be audited is 98% and it should be aimed to be 100%.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| **Code Structure and Formatting**        | The codebase contracts are well-structured and formatted. It inherits from multiple components, adhering for-example The Ocean.sol contract follows a modular structure with clear separation of concerns into Core contract logic, inherited OpenZeppelin standards, and imported supporting contracts.Functions are grouped thematically and formatted uniformly with visibility, modifiers, and error handling.efficiency.                                                                                                                                                                                                                                                       |
| **Strengths**                            | Comprehensive unit tests,Utilization of Natspec                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |

## Systemic & Centralization Risks

The analysis provided highlights several significant systemic and centralization risks present in the Shell v3 protocol.
These risks encompass concentration risk in Ocean, Curve2PoolAdapter, CurveTricryptoAdapter risk and more, third-party
dependency risk, and centralization risks arising. Additionally, the absence of fuzzing and invariant tests could also
pose risks to the protocol’s security.

Here's an analysis of potential systemic and centralization risks in the contracts:

### Systemic Risks:

1. **No having, fuzzing and invariant tests could open the door to future vulnerabilities**.

2. Large linked transactions handled by doMultipleInteractions in Ocean contract could fail or revert half-complete,
   leaving token balances in an inconsistent state across multiple users/accounts.

3. Flash loan type attacks are possible via doMultipleInteractions if interactions are not properly validated to prevent
   actions like wrapping assets, selling them at a discounted price, then unwinding.

4. Flash loan attacks may be possible by calling primitiveOutputAmount multiple times Curve2PoolAdapter.sol contract
   with unrealistically large trade amounts.

5. The CurveTricryptoAdapter contract approves an unlimited amount of tokens for spending by ocean and primitive. This
   can be risky if these allowances are not required or if the recipient contracts are compromised.

6. Managing all token state in one contract makes it a bigger target and increases risk of a compromise compromising the
   whole system.

### Centralization Risks:

1. The owner/DAO has significant control over the Ocean contract through functions like changing the unwrap fee. If the
   owner/DAO becomes centralized, it could enable censorship or modifications against users' interests.

2. The owner/DAO has control over whitelisting new tokens/pools and could censor certain assets in Curve2PoolAdapter
   contract. This centralizes which integrations are available.

3. Central points of failure like the Ocean contract increase fragility - a single point of failure can cascade.

**Properly managing these risks and implementing best practices in security and decentralization will contribute to the
sustainability and long-term success of the Shell protocol.**

## Conclusion

In general, The Shell protocol presents a well-designed architecture for managing generative NFT collections with its
set of core contracts, we believe the team has done a good job regarding the code. However, the analysis revealed some
security and decentralization risks that warrant attention. Chief among these is over-reliance on centralized admin
accounts for critical functions. Additionally, it is recommended to improve the documentation and comments in the code
to enhance understanding and collaboration among developers and auditors. It is also highly recommended that the team
continues to invest in security measures such as mitigation reviews, audits, and bug bounty programs to maintain the
security and reliability of the project.


### Time spent:
30 hours