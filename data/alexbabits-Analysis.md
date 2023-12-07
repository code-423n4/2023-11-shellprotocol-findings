# Overview
Within the context of this audit, the goal of Shell protocol is to make external protocols compatible with the Ocean through adapter primitives. These adapters take in a financial primitive with respect to the Ocean, and handle standardizing the computation of inputs and outputs so that they can be used seamlessly in the context of the Ocean. Financial primitives are things like AMMs, lending pools, NFT markets, and so on. The Shell protocol is on Arbitrum.

Traditionally, primitives do the business logic of mathematical calculation of inputs and outputs and the actual accounting or movement of tokens. This causes users who wanted to participate in DeFi to execute multiple transactions across multiple primitives for a single intended end result or action.

* There are sequencer-adapter models that dApps like 1inch.io and Matcha.xyz use, but every protocol requires a custom adapter and the primitives are lone wolves.
* There are vault-router models that dApps like Balance v2 and Uniswap v3 use, which are an improvement and allow AMM developers to focus just on the business logic, but they can only compose with AMMs and not all primitives.
* There is the Ocean model that takes inspiration for both, allowing gas savings from the vault-router model and the ability to compose with any kind of primitive.

The Ocean solves the problem of scattered and non-standardized primitives by handling all the accounting movement (burns, mints, transfers) of tokens via its internal ledger, letting the primitives focus solely on the mathematical business logic of calculating amount B for token B, given token A, amount A, and token B. By having standardized compute functions for primitives, and by putting all the tokens into a single ERC1155 ledger, Shell protocol has boiled down the commonality between any primitive and has separated the concerns of accounting and business logic.


### Scope
There have been multiple Code4rena audits of the Shell Protocol. This one is focused on the Ocean and the Adapters.

Scoped Files:

* `Ocean.sol` - Handles the accounting or "movement" of tokens.
* `OceanAdapter.sol` - Helper file for the Curve pool adapters. The adapters force Curve to share the same exact accounting logic of the Ocean as any other primitive that we want to adapt into the Ocean.
* `Curve2PoolAdapter.sol` - Used to access Curve's output amount for deposits, withdraws, and swaps. The pool can be found at: https://curve.fi/#/arbitrum/pools/2pool/deposit
* `CurveTricryptoAdapter.sol` - Used to access Curve's output amount for deposits, withdraws, and swaps. The pool can be found at: https://curve.fi/#/arbitrum/pools/tricrypto/deposit


Relevant out of scope files:

* `BalanceDelta.sol` - Handles the temporary balances that are stored in memory during a set of multiple interactions. Useful for understanding how the balances in memory are handled.
* `Interactions.sol` - Contains the Interaction struct and InteractionType enum, as well as the interface for the Ocean's `do*` and `forwardDo*` functions. Useful for understanding interactions and their types in detail.
* `OceanERC1155.sol` - The ERC1155 ledger for the Ocean that handles the internal accounting. It also handles registering new primitive tokens, calculating the ocean ID, doing mints, burns, and safe transfers to modify the balances in storage. 
* `MaliciousPrimitive.sol` - Example of a malicious primitive that sets a small output amount to be given to a user and a large input amount to be taken from a user.
* `MockPrimitive.sol` - Example primitive that uses interactions (on behalf of the malicious primitive) to determine input/output amounts.



# Codebase Quality & Audit Approach
The codebase is well documented and structured logically. For incoming developers, consider having more strict language and background education on the adapters and other things unique to the protocol, as those were the hardest to grasp as a novice. Below is my audit approach.

1. Compile and run their test suite.
2. Read all the documents, the whitepaper, and past audits.
3. Asked myself questions about the protocol and went looking for answers. The white paper was extremely useful and relevant to the code base. Most questions could be solved by carefully looking through the codebase and whitepaper. Another powerful resource was their test suite in foundry and hardhat, so I could see how the adapters and the Ocean were supposed to work. 
4. Asked the developers questions when I didn't understand something, otherwise re-reading the white paper and codebase worked.


Notable questions asked during the audit:

* What malicious things can the primitives or adapters do to poison the accounting of the ocean, if anything? They just return a uint256 output amount, which could be malicious but this doesn't seem to affect any other aspect of the ocean's ledger. 

* What if the userAddress gets blacklisted during or after a set of interactions? Cannot be blacklisted during a set of interactions since it is a single atomic transaction. If blacklisted afterwards, then they cannot receive their balance via an unwrapping because safeTransfer will fail, but I think this will not damage Ocean's internal accounting. 

* What if we send ETH value and tokens at the same time in a `doInteraction()` or `doMultipleInteraction()`? In the former, because there is an if-else statement, the interaction specifics doesn't actually matter, so if a value of ETH is sent, then it doesn't even bother to look at the interaction part. For multiple interactions, it can handle an ETH value sent and token interactions.

* Can anything bad happen if passing in invalid interactionType or address for the bytes32 interactionTypeAndAddress? An interactionType beyond the enum will revert in the `_getSpecifiedToken()` check. Any address can be passed for the address parameter, but this just tells which token contract or primitive to interact with. So unless the primitive or token itself can somehow poison ocean, this parameter is also safe.

* Can anything malicious happen via the bytes32 metadata for an interaction? In the case of the curve pool adapters, the metadata is minimumOutputAmount, which gets cast from bytes32 to uint256, and is used as a slippage metric. Notably, if this is set to 0 and the output amount is 0, the check will pass. It is technically arbitrary but handled by the primitives which ultimately just return amounts.

* Is calculating the ocean ID for an ERC721 based on the metadata on the fly during `doInteraction()` a problem? If an invalid bytes32 value that is supposed to be cast to the token ID of the NFT is invalid, then it should simply return an invalid ocean ID and revert during `_executeInteraction()`.

* What if there are duplicate tokenId's in the BalanceDelta struct? Answered directly in the code comments in `BalanceDelta.sol`. Any duplicates will always have a delta of 0 because the iteration will only ever look/grab the first tokenID.

* What if we provide two identical tokenIds and somehow get both to have a different delta, knowing that only the first token will be operated on? This could cause something bad that may be worth taking a look, but I didn't understand how that would be possible.

* Are flash mints and flash loans possible? If so, what kinds of malicious things can happen during a flash mint or loan? Yes they are, but flash loans aren't traditional flash loan I think. In detail answer later.

* How can we mess up the balance deltas in memory? We can technically do any whacky things we want in the temporary balance deltas, but "judgement day" comes after a set of interactions via mints and burns, and should always revert if the actual balances in storage attempt to become less than 0.

* Can we make the _balances mapping in storage have any value of less than 0? It appears not, sanitized through uint256 and with checks for Balance >= amount before subtractions. (Where amount is always positive).

* Can we make the calculated ocean ID incorrect or duplicate with different input values via the abi.encodePacked? No, because an address is always 20 bytes and the tokenId uint should always be unique. Unless there is a way for the tokenAddress for two different tokens to be identical, then this could pose an issue. We cannot mix and match or swap the tokenAddress or tokenId in any capacity.


# Mechanism Review
There is a nice separation of concerns between the Ocean files and the primitive adapters. The primitives are tasked with computing output/input amounts, while the ocean handles all the transfers, mints, and burns. 


### Invariants 
* Balances on the ocean's ledger cannot be negative. (Not to be confused with the temporary balances tracked during `doMultipleInteractions()` which can have positive and negative deltas during interactions).
* A user's balances should only move with their permission
* Fees should be credited to the Ocean owner's ERC-1155 balance
* Calculation of wrapped token IDs is correct and collision resistant
* Calls to the Ocean cannot cause the Ocean to make external transfers unless a doInteraction/doMultipleInteractions function is called and a wrap or unwrap interaction is provided.
* Calls to the Ocean cannot cause it to mint a token without first calling the contract used to calculate its token ID.
* The Ocean should conform to all standards that its code claims to (ERC-1155, ERC-165), except 
* The Ocean does not support rebasing tokens, fee on transfer tokens
* The Ocean ERC-1155 transfer functions are secure and protected with reentrancy checks
* During any `do*` call, the Ocean accurately tracks balances of the tokens involved throughout the transaction.
* The Ocean does not provide any guarantees against the underlying token blacklisting the Ocean or any sort of other non-standard behavior
* Ocean must guarantee that a ledger entry for a malicious token cannot impact ledger entries for good tokens. Ocean's ledger cannot be poisoned by misbehaving token.


### Flow of doMultipleInteractions()
1. Creates temporary balances array setting balances for each token in memory to 0 for this set of interactions. If ETH is sent it immediately accounts for that and increases Alice's wrapped ether amount in memory.
2. Begins iterating through the array of interactions on behalf of the user address with the associated input and output token ids
    * unpacks the interactionTypeAndAddress data to get the interactionType enum and address. The address is either the primitive or the token contract address. 
    * With this information, we can grab the corresponding token's Ocean ID via `_getSpecifiedtoken()`.
    * Now that we have the specified token, we can grab the specified amount. If it is uint256 max, then we just get the current balance delta in memory, otherwise we use the specifiedAmount member in the interaction struct.
    * With the specified amount and specified token now known, we can actually execute the interaction with `_executeInteraction()`. This will execute whatever specified interaction type was passed, and we expect a wrap, unwrap, or computation of an input or output amount to occur. 
        * If the input or output amount changes, we modify the temporary balances accordingly in memory.
        * In the case of wrapping/unwrapping, a safeTransfer is done after determining the proper amount.
3. Once the loop finishes iterating through and executing all the interactions and modifying the temporary balances whenever necessary, "judgement day" begins where any mints/burns are done based on the ending balances in memory. 
    * Mint/burn IDs and amounts arrays are made via `createMintAndBurnArrays()`, and populated with proper amounts via `_copyDeltasToMintAndBurnArrays()`
    * Mints/burns are executed via the Oceans 1155 ledger, changing the balances in storage. Notably, this is the only storage write for the entire set of interactions, and these balances cannot be negative unlike the temporary balances.


### Swapping 100 DAI for 100 USDC
1. Alice (or a forwarder on behalf of Alice) will approve the Ocean for her tokens, and then initiate `doMultipleInteraction()` with 3 interactions.
2. Alice prepares an interaction of interactionType wrapErc20, specifiying DAI as the inputToken and inputAddress, and the specifiedAmount in decimals 1e18 regardless of the decimals. This allows the Ocean to safe transfer from Alice to the Ocean. This mints Alice 100 shDAI to the balance delta in memory. 
3. Alice prepares an interaction of interactionType computeOutputAmount, specifiying the AMM primitive as the external contract, and the desired tokens and amounts. The AMM swaps the shDAI for shUSDC.
4. Alice prepares an interaction of interactionType unwrapErc20, specifiying shUSDC to be withdrawn from the ocean. This burns the 100 shUSDC from the balance delta memory, and the Ocean transfers the 100 USDC to Alice.


### Flash minting 1_000_000 shDAI
Alice prepares an interaction of interactionType computeOutputAmount, specifying an AMM primitive as the external contract with shUSDC as the input token and shDAI as the output token for a specifiedAmount of 1e24 (1e6 * 1e18). This makes the temporary balances of Alice -1 million shUSDC and +1 million shDAI. Notably, in future interactions she could only unwrap however much the Ocean has backing the shUSDC if she wanted to also do a flash loan. If her starting balance of shUSDC in the Ocean ledger storage is 0, then the ending temporary balance would have to be at least 0, otherwise the whole transaction would revert.


### Flash loaning 10_000 DAI
1. Alice does a flash mint as previously described, so she has a positive balance of 10_000 shDAI and negative balance of 10_000 shUSDC.
2. Alice unwraps the shDAI which gets deducted in the temporary balances, and 10_000 DAI goes to the ERC20 ledger for Alice.
3. Alice can call a primitive that she controls with a `Compute*` interactionType so the primitive that does the flashloan is still invoked.
4. The primitive can then do further interactions with the flash loaned tokens.
5. Alice has to wrap the 10_000 DAI back into shDAI, and swap back for 10_000 shUSDC to repay her negative 10_000 shUDSC balance in order for `doMultipleInteractions()` to successfully happen.

Because `doMultipleInteractions()` happens atomically, everything must be done within the context of interactions, and the ending balances in storage must be properly rectified for a successful flash loan. 


# Risks (Centralization, External, Systematic) & Recommendations

* Centralization risk in `Ocean.sol` via onlyOwner modifier with access to changing the fees arbitrarily. There is also no delay on changing fees. The code documentation properly states that the governance structure must handle time lock or mechanism for managing fee changes.

* External risk via Curve. The adapters compute the rawOutputAmount based on the curve pool. If they ever change how decimals are handled or their protocol fails, the adapter will also be rendered useless. This external risk is a peripheral concern but should be noted. Also, Curve uses Vyper instead of Solidity in their contracts. Regarding the Curve tricrypto pool for Arbitrum, https://curve.fi/#/arbitrum/pools/tricrypto/deposit, the website states that the pool may be at risk of being exploited, and they recommend exiting this pool. Tweet here: https://twitter.com/CurveFinance/status/1685925429041917952. Therefore, this may not be the best optics to display how adapters work, to have a connection with the tricrypto pool, even if the chance of tragedy is low, the sentiment could scare people.

* Systemic risk via putting all of the eggs in one basket, or all of the shells in one Ocean ledger. If something critical fails with the ocean or ocean ledger, then all the primitives connected are effected. There is no upgradability or pausability, which is a tradeoff between absolute immutability and potential centralized safety backdoors.

* Compiler risk via solidity compiler 0.8.20. I am unsure if this is an issue but it is worth noting. The PUSH0 opcode introduced in 0.8.20 is not supported on Arbitrum. (This is a more gas-efficient way to push 0 on top of the stack). This may create compatibility issues, where contracts compiled with these versions may deploy but fail to function or return incorrect values. A solution if it is of concern is to downgrade all contracts to 0.8.19.

* Dependency Note: OpenZeppelin version 4.8.1 appears to be a solid choice, especially because in version 5.0.0 they removed `.isContract()`, which is used in the `OceanERC1155.sol` while doing TransferAcceptance checks. It appears `isContract()` is used appropriately.

* Primitive Security Concerns: The user experience and specific security concerns are passed onto the primitives rather than the Ocean itself. While this keeps the Ocean pristine, if primitives integrating with Ocean do so poorly it can lead to a bad taste for the user for the Ocean in general. The primitives are the ones that need to be selective in which tokens they can accept, and make sure to properly understand how the Ocean works. Because a lot of the security concerns are passed along to the adapters/primitives, make sure to have an overbearing amount of education and developer docs for building on top of Ocean.

* Current security approach: They have had past audits with Trail of Bits, a bug bounty program on Immunefi, have participated in multiple Code4rena audits, and they have a Forta bot that can monitor wraps/unwraps/swaps/deposits/withdraws over a threshold amount.

* Ideally, the test suite would make sure to include all the invariants and exploration of flash mints and flash loans along with Echidna fuzzing. And generally mix and matching longer strings of `doMultipleInteractions()` interactions during tests could be useful.


# References
- Shell Protocol Github: https://github.com/Shell-Protocol/Shell-Protocol
- Shell Protocol Docs: https://wiki.shellprotocol.io/getting-started/overview
- 4naly3er Report: https://github.com/code-423n4/2023-11-shellprotocol/blob/main/4naly3er-report.md
- Bot Report: https://github.com/code-423n4/2023-11-shellprotocol/blob/main/bot-report.md
- Shell v2 Whitepaper: https://github.com/Shell-Protocol/Shell-Protocol/blob/main/Ocean_-_Shell_v2_Part_2.pdf
- Trail of Bits Audit: https://github.com/trailofbits/publications/blob/master/reviews/ShellProtocolv2.pdf
- v3 Audit Report: https://docs.google.com/document/d/1HJpomEsY4dAyXsQ3MF27YFaX74QS4EE05f76ikLg25c/edit
- Past Code4rena Audit for Proteus: https://code4rena.com/reports/2023-08-shell
- Forta Bot for Shell: https://app.forta.network/bot/0x7f9afc392329ed5a473bcf304565adf9c2588ba4bc060f7d215519005b8303e3
- OpenZeppelin Releases: https://github.com/OpenZeppelin/openzeppelin-contracts/releases

### Time spent:
35 hours