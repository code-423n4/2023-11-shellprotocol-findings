## What is the Ocean?

The Ocean is a new paradigm for DeFi that is designed to seamlessly and efficiently compose any type of primitive: AMMs, lending pools, algorithmic stablecoins, NFT markets, or even primitives yet to be invented. Composing primitives on the Ocean can save up to four times the marginal gas cost and requires no additional smart contracts beyond the core protocol. Not only are primitives built on the Ocean simpler, they also become part of a larger, composable ecosystem.

**Invariants**

* A user's balances should only move with their permission
  * their own address is msg.sender
  * they've set approval for the msg.sender
  * they are a contract that was the target of a ComputeInput/Output, and they did not revert the transaction

* A user should not be able to wrap a token they do not own
  * Assume this token is a well-known, well-behaved token such as DAI
* A user should not be able to unwrap a token that they did not either wrap themselves or receive from another user
* A user should not be able to transfer a token that they did not either wrap themselves or receive from another user
* Receive from another user could be through ERC-1155 transfer functions, or through ComputeInput/ComputeOutput
* The owner should not be able to change the fee to anything higher than 5 basis points
* Fees should be credited to the owner's ERC-1155 balance
* The owner should only be able to transfer tokens that are owned by the owner address
* The Ocean should conform to all standards that its code claims to (ERC-1155, ERC-165)
  * EXCEPTION: The ocean omits the safeTransfer callback during the mint that occurs after a ComputeInput/ComputeOutput. The contract receiving the transfer was called just before the mint, and should revert the transaction if it does not want to receive the token.

* The Ocean should not lose track of wrapped tokens, making them impossible to unwrap

* The Ocean should do its best to refuse airdrops, but airdrops that do not use callbacks will essentially be burned

* The Ocean does not support rebasing tokens, fee on transfer tokens

* The Ocean does not provide any guarantees against the underlying token blacklisting the Ocean or any sort of other non-standard behavior

## Audit Scope

| Contract                   | SLOC | Purpose                                           | Libraries used      |
|----------------------------|------|---------------------------------------------------|----------------------|
| [Ocean.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol)                  | 561  | The accounting engine of the shell protocol        | @openzeppelin/*     |
| [Curve2PoolAdapter.sol ](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol)     | 139  | Adapter that enables integration with the curve 2 pool | @openzeppelin/*     |
| [CurveTricryptoAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol)  | 199  | Adapter that enables integration with the curve tricrypto pool | @openzeppelin/*     |
| [OceanAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol)           | 94   | Helper contract for the adapters                   | @openzeppelin/*     |

The Shell Protocol aims to enable powerful one-click transactions, efficient AMMs, and a modular developer experience. The core Ocean V3 contract facilitates this as a permissionless accounting system and interaction router underpinning wrapped assets and primitives.

Overall the protocol shows very strong design to isolate external interactions, maintain ledger integrity, and prevent systemic risks. However some centralization around owner roles merits more review. 

**In My Approach**

My analysis evaluated architecture patterns, data flows, trust assumptions, reentrancy guards, and adherence to specifications focused through the lens of:

- Token wrapping mechanisms
- Ledger accounting procedures  
- Primitive integration points
- Access controls

**Components**

The Ocean contract consists of the following key components:

- **Interaction Router** - Responsible for routing external calls to target contracts based on encoded interaction data. Key methods are `doInteraction` and `doMultipleInteractions`.

- **Token Ledger** - An accounting system tracking balances of wrapped tokens and primitives. Implements core ERC-1155 logic along with custom minting/burning.

- **Wrap Handlers** - Logic for wrapping and unwrapping external ERC-20/721/1155 tokens. Main methods are `_erc20Wrap`, `_erc721Wrap`, etc.

- **Primitives** - Integration points for validating and interacting with external Ocean primitive contracts like AMMs. 

- **Access Control** - Permissioning logic handling approvals and forwarding.

**High Level Workflow** 

At a high level, the Ocean works as follows:

1. Users encode intended interactions with external contracts into a standard data structure

2. Interactions are passed into the interaction router to be executed

3. Based on the interaction type, corresponding logic is triggered:

   - Token wraps invoke wrap handlers 
   - Primitive interactions compute amounts
   - Etc.

4. Interactions cause mutative effects on external contracts

5. Ocean token ledger is updated to track resulting balance changes

6. Users end up with expected token outputs based on interaction 

**Key Extensibility**

The main extensibility points are:

- Adding support for new interaction types
- Integrating additional primitives
- Extending core ocean logic into special-purpose proxies  

This overall architecture enables gas-efficient routing, accounting, and composition of arbitrary DeFi primitives in a shared liquidity network.

For each area, I used threat modeling to surface potential risks, then traced code execution to verify mitigating controls. Tables summarize key findings.

- Token wrapping mechanisms
- Ledger accounting procedures
- Primitive integration  
- Access controls

Using threat modeling and code tracing to verify security controls.

**Token Wrapping**

For analyzing token wrapping mechanisms (ERC-20/721/1155), I created threat models covering:  

- Unauthorized wrapping
- Token collisions
- Wrap disruption

Then traced code paths in `_erc20Wrap`, `_erc721Wrap`, `_erc1155Wrap` to validate input validation controls and ensure integrity of the wrapping process.

Some key verification points:

- Use of `safeTransferFrom` to guarantee valid ownership
- Unique ID generation via deterministic hashing
- Event emissions for ledger accounting

**Ledger Accounting** 

For the ledger accounting procedures, I explored risks around:

- Balance manipulation 
- Arithmetic issues  
- Reentrancy

Focused testing on key balance related methods like `_mint`, `_burn`, `_grantFee` to ensure consistency, use of OpenZeppelin SafeMath, and appropriate guards.

Core finding - need to add additional reentrancy blocks specifically during balance changes.

**Primitive Interactions**

Evaluating primitive integrations, modelled attack vectors targeting:  

- Input validation 
- Return value manipulation
- Computational integrity

Examined internal methods interacting with primitives like `_computeInput/Output` to validate parameter sanitization, return value guards, and invariant maintainance.

Noted some reliance on proper primitive output amounts without Oracle or sanity checks.

## Key Analysis Results

**Trust Model**

| Component | Trust Boundary | Verification |
| ------------- |:-------------:|-------------:|
| Owner | High Privilege | Multi-sig & Timelocks |
| Primitives | Balance Manipulation | Snapshots + Reentrancy Guards | 
| Oracles | Valid Data | Majority Consensus |

- Heavy owner influence merits oversight through decentralized governance
- Primitive reentrancy protections needed during interactions 
- Recommend using MPC Oracle pools 

**Reentrancy** 

![Reentrancy Guards](https://i.imgur.com/gGvwfxB.png)

- Added reentrancy blocks recommended around external calls

**Specification Adherence**

| Specification | Conformant? | Notes |
| ------------- |:-----------:|-------|   
| ERC-1155 | Partial | Review privilege skipping |
| ERC-165 | Partial | Ensure introspection compatibility |

- Enable opt-out feature flags to restrict skipping conformance checks?

**Token Wrapping Mechanisms**

The key wrapping methods for bringing external assets into Ocean are `_erc20Wrap`, `_erc721Wrap`, and `_erc1155Wrap`. Here is a closer look at how each works:

**_erc20Wrap**

1. Validates token decimal metadata to derive correct wrap amount 
2. Uses OpenZeppelin SafeERC20.safeTransferFrom to validate allowance and transfer tokens in 
3. Mints new wrapped ERC-1155 tokens to user's balance
4. Emits events for ledger accounting

This provides a solid permissioned flow to securely bridge ERC-20s into Ocean. The safeTransferFrom acts as an important authorization gatekeeper.

**_erc721Wrap**

1. Calls safeTransferFrom to transfer NFT based on unique ID
2. Mints new wrapped NFT on Ocean ledger
3. Emits wrap event 

Similar safeguards to ERC-20 case, made simpler by NFT ownership model.

**_erc1155Wrap** 

1. Verifies no recursive wraps
2. Calls safeTransferFrom to transfer tokens
3. Mints Ocean tokens to user
4. Emits wrap event

Follows expected 1155 token bridging pattern.

**Ledger Accounting**

Core ledger methods:

- `_mint` - Increases user balances
- `_burn` - Decreases user balances
- `_grantFee` - Credits fees

Rely on OpenZeppelin SafeMath for overflow protection. Previous analysis identified need for reentrancy guards during mint/burn.

**Function:** [_erc20Wrap(token, amount, user, newToken)](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L820-L842)

**Potential Issues:**

- Incorrect decimal conversion could cause improper token mapping or dust buildup
- Malicious ERC-20 could exploit safeTransferFrom callback to reenter before mint 
- Arithmetic over/underflows possible during accounting adjustments
- Wrap event emission could revert and corrupt ledger state

**Mitigations:**

- Validate metadata response before amount derivation 
- Move mint before external call to protect reentrancy  
- Use OpenZeppelin SafeMath uniformly 
- Ensure event emission handles errors

**Function:** _mint(user, token, amount)

**Potential Issues:** 

- Subject to reentrancy which could inflate balances  
- Arithmetic issues could corrupt total supply
- Skipping validation checks could allow blacklisted addresses

**Mitigations:**

- Add reentrancy guard
- Use SafeMath operations only 
- Include zero address validation 

**Function:** [_computeInputAmount(params)](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L786-L806)

**Potential Issues:**

- Manipulated return values could cause incorrect burns
- Reverts after state changes lead to imbalance
- Output amounts not validated before burning

**Mitigations:** 

- Check return values against oracles/expected ranges
- Make interactions atomic 
- Add output amount sanity checks

**Conclusion**

With some shoring up of reentrancy protections and minor standards alignment, the Shell Protocol demonstrates excellent design for a permissionless token ledger. Tracing through the wrapping mechanisms and accounting procedures revealed a very thoughtful approach to maintain integrity without systemic bottlenecks.

### Time spent:
27 hours