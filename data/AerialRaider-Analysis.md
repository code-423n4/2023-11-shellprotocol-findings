Audit of the Shell Protocol has identified several key areas that require attention to enhance security and functionality. Here's a summary of the findings:

Minting Control in Ocean Contract: 
It's essential to ensure that the Ocean contract does not mint any tokens without first calling the contract used to calculate its token ID. This control mechanism prevents unauthorized token creation and ensures token IDs are valid and correctly calculated. Risk Level: Medium.

Handling of Rebasing and Fee-on-Transfer Tokens: 
The interaction of rebasing tokens and fee-on-transfer tokens with the Ocean contract poses challenges. The contract needs to address these types of tokens' unique behaviors to ensure accurate balance tracking and transaction processing. Risk Level: Medium.

Balance Tracking in Interactions: 
During doInteraction or doMultipleInteractions calls, it's critical for the Ocean contract to accurately track and update the balances of the tokens involved. This ensures the integrity of transactions and maintains correct accounting within the contract. Risk Level: Medium.

Fee Allocation to Ocean Owner: 
Fees generated within the Ocean contract should be correctly credited to the Ocean owner's ERC-1155 balance. This involves ensuring the fee mechanisms and calculations are precise and that the fees are properly allocated and recorded. Risk Level: Medium.

Validations in checksprimitiveOutputAmount Function: 
Implementing rigorous validations in the checksprimitiveOutputAmount function will enhance security and reliability. This enhancement will prevent potential errors or vulnerabilities in the output amount computation process. Risk Level: Medium.

Integrity of Token Wrapping/Unwrapping in Curve2PoolAdapter: 
Ensuring the integrity of the token wrapping and unwrapping processes in the Curve2PoolAdapter contract is crucial. This involves verifying that these processes are secure, error-free, and function as intended, especially concerning token conversions and liquidity pool interactions. Risk Level: Medium.

### Time spent:
20 hours