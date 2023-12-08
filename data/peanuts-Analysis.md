### Summary of the protocol

- User can deposit any types of tokens into the Ocean to get the corresponding shellTokens (shTokens, sh- prefix, 18 decimal places). Eg, users can deposit 100e6 USDC tokens to get 100e18 USDC tokens.
- The shellTokens is tabulated using the ERC1155 specification. This means that there will be a one huge ledger tracking every one's shTokens
- If users wants to swap their shTokens, adapter primitives will be used to interact with protocols. For example, if a user wants to swap their shUSDC tokens to shWBTC, they can do so using the external primitives.
- Users can also bundle up interactions instead of having to call a function one at a time
- At this point, there are 9 interaction types.

```
enum InteractionType {
    WrapErc20,
    UnwrapErc20,
    WrapErc721,
    UnwrapErc721,
    WrapErc1155,
    UnwrapErc1155,
    ComputeInputAmount,
    ComputeOutputAmount,
    UnwrapEther
}
```

- The intention is to create a huge ledger book and a common ground to facilitate transactions between any types of tokens (ERC20, ERC721, ERC1155), as well as reduce gas fees. For example, if someone wants to buy an NFT, instead of having to go to a marketplace, creating a transaction, checking for approvals, deducting balance and transferring tokens, users can just trade shERC20 for shERC721 tokens.

### Approach taken in evaluating the codebase

| Step | Approach                     | Details                                                                                       | Comments                                                           |
| ---- | ---------------------------- | --------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| 1    | Reading the Documentation    | [Shell Protocol Wiki](https://wiki.shellprotocol.io/how-shell-works/the-ocean-accounting-hub) | Clear and concise, purpose is easily understood                    |
| 2    | Understanding the Whitepaper | [Whitepaper](https://shellprotocol.io/static/Ocean_-_Shell_v2_Part_2.pdf)                     | Great examples, in-depth explanation, building on top of the docs  |
| 3    | Protocol Walkthrough         | [Shell App](https://app.shellprotocol.io/trade)                                               | Great UI, nice to see the frontend to make sense of things         |
| 4    | Downloading the Code         | [Github](https://github.com/code-423n4/2023-11-shellprotocol/tree/main)                       | Build is fine                                                      |
| 5    | Manual Code Review           | [Github](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol)   | Started with the biggest nSLOC, Ocean.sol, matching docs with code |
| 6    | Manual Analysis              | [Github](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol)   | Started writing the Analysis report, and invariant testing         |
| 7    | Discord                      | -                                                                                             | Read through the discord discussions to confirm understanding      |
| 8    | Submissions                  | -                                                                                             | Submit Medium Findings, QA report and Analysis Report              |

### How the Code Works

1. Wrapping Tokens

- User has 100e6 USDC tokens
- He calls `doInteraction()` with the specific ERC20Wrap interaction. 
- User will get 100e18 shUSDC tokens (ignoring fees)
- The 100e6 USDC tokens will be transferred into the Ocean.sol contract

2. Unwrapping Tokens

- User has 100e18 shUSDC tokens
- He calls `doInteraction()` with the specific ERC20Unwrap interaction. 
- User will get 100e6 USDC tokens (ignoring fees)
- The contract will have transferred 100e6 USDC tokens to the user

3. Swapping Tokens (Swapping 100e18 shUSDC to 100e18 shUSDT)

- User has 100e18 shUSDC tokens and wants to swap to 100e18 shUSDT tokens. He needs to rely on a primitive to interact with AMMs
- He calls `doInteraction()` with the specific ComputeOutputAmount interaction. 
- A temporary ledger will be created for the Primitive.
- Primitive will mint 100e18 shUSDC tokens
- Primitive will unwrap 100e18 shUSDC tokens and get 100e6 USDC tokens (The USDC tokens have been transferred from the Ocean contract to the Primitive contract)
- Primitive swaps the tokens on an AMM (eg Curve) and gets back 99.9e6 USDT tokens (slippage and fees on curve)
- Primitive will wrap the 99.9e6 USDT tokens to 99.9e18 shUSDT tokens (USDT tokens have been transferred from Primitive contract to Ocean contract)
- Primitive will burn 99.9e18 shUSDT tokens
- User will burn 100e18 shUSDC tokens and mint 99.9e18 shUSDT tokens


### Codebase quality analysis and review

| Contract          | Line | Function                   | Comments                                               | Questions/Inconsistencies to Review                                           |
| ----------------- | ---- | -------------------------- | ------------------------------------------------------ | ----------------------------------------------------------------------------- |
| Ocean             | 196  | changeUnwrapFee            | Unwrap fee must be above MIN_UNWRAP_FEE_DIVISOR (2000) | No upper limit, its ok, just lesser fees. onlyOwner centralization            |
| Ocean             | 1159 | calculateUnwrapFee         | used in erc20, erc1155 and ether unwrap                | Maximum fee is 0.05% for every unwrap                                         |
| Ocean             | 864  | erc20Unwrap                | unwrap ERC20 token, can be any token?                  | truncatedAmt goes to protocol as well                                         |
| Ocean             | 820  | erc20Wrap                  | wrap ERC20 token, can be any token?                    | mirror of unwrapping, uses determineTransferAmt                               |
| Ocean             | 1123 | convertDecimals            | convert shERC decimals to ERC decimals                 | 11 variations, checks below                                                   |
| Ocean             | 1123 | determineTransferAmount    | used in erc20 wrap, from x decimals to 18              | uses convertDecimals as well, but convert from 18 to x?                       |
| Ocean             | 889  | erc721Wrap                 | wrap ERC721 tokens, no fees, specifiedAmt must be 1    | TokenId is a user input, any reentrancy? Not really applicable                |
| Ocean             | 903  | erc721Unwrap               | wrap ERC721 tokens, no fees                            | TokenId is a user input, can withdraw any token from collection?              |
| Ocean             | 920  | erc1155Wrap                | wrap ERC1155 tokens to shTokens, no fees               | TokenId is a user input                                                       |
| Ocean             | 955  | erc1155Unwrap              | unwrap ERC1155 tokens, pays a fee                      | TokenId is a user input, can withdraw any token from collection?              |
| Ocean             | 210  | doInteraction              | payable, sets msg.sender                               | 9 forms of interaction type                                                   |
| Ocean             | 229  | doMultipleInteractions     | payable, sets msg.sender                               | Chains the interaction type, reentrancy?                                      |
| Ocean             | 256  | forwardedDoInteraction     | payable, sets approved forwarder                       | Same as doInteraction, but lets an approved forwarder do the interaction      |
| Ocean             | 256  | increaseBalanceOfPrimitive | Increases balance of primitive temporarily             | This is to make sure that external primitives can interact with Ocean         |
| OceanAdapter      | 55   | computeOutputAmount        | user provides inputAmt and primitive compute outputAmt | A external adapter primitive, inherited by Curve2PoolAdapter.sol              |
| OceanAdapter      | 108  | calculateOceanId           | Provides the tokenAddress and tokenId                  | tokenId == 0 for ERC20, what if two address tokens?                           |
| Curve2PoolAdapter | 102  | wrapToken                  | wraps a ERC20 token, calls doIntearction               | Curve2PoolAdapter becomes the msg.sender in doInteraction?                    |
| Curve2PoolAdapter | 142  | primitiveOutputAmount      | swaps/addliquidity/removeliquidity from curve          | Slippage is checked, primitive helps to swap shTokens? Does it affect ledger? |
| Curve2PoolAdapter | 118  | wrapToken                  | similar to Curve2PoolAdapter, but with ETH integration | zToken is ETH, interactionTypeAddress sets to 0 (indicate ETH Wrapping)       |

Manual Invariant Analysis (Truncation/Rounding)

Check calculateUnwrapFee

- 1e18 / 2000 = 5e14
- 5e14 / 1e18 = 0.0005
- 0.05% of 1e18 is 5e14.

Check convertDecimals (Unwrapping)

1. From 18 decimals to lower decimals

- eg 100 USDC
- 100e18 shUSDC -> 100e6 USDC
- Fees = 5e16, amount to convert = 9.995e19
- convertedAmount = (else clause)
- shift = 1e12, convertedAmount = 99.95e6, truncatedAmount = 0

2. From 18 decimals to 18 decimals

- eg 100 USDT
- 100e18 shUSDT -> 100e18 USDT
- Fees = 5e16, amount to convert = 9.995e19
- convertedAmount = (first if clause) = 99.95e18, truncatedAmount = 0

3. From 18 decimals to 30 decimals

- eg 100 (asd Token)
- 100e18 shASD -> 100e30 shASD
- Fees = 5e16, amount to convert = 9.995e19
- convertedAmount = (else if clause)
- shift = 1e18, convertedAmount = 99.95e30, truncatedAmount = 0

4. Bonus - from 18 decimals to 1 decimal (try to get truncatedAmount != 0)

- 100e1 asdToken
- Fees = 5e16, amount to convert = 9.995e19
- shift = 1e17, convertedAmount = 99.9, truncatedAmount = 0.5e18

Check convertDecimals (Wrapping) via determineTransferAmount

1. From 18 decimals to 6 decimals

- eg USDC -> shUSDC, 100e6
- \_convertDecimals(18,6,100e18)
- transferAmount = 100e6, dust = 0

2. From 18 decimals to 18 decimals

- eg USDT -> shUSDT, 100e18
- \_convertDecimals(18,18,100e18)
- transferAmount = 100e18, dust = 0.

3. From 18 decimals to 30 decimals

- \_convertDecimals(18,30,100e18)
- transferAmount = 100e30, dust = 0

4. Bonus: From 18 decimals to 2 decimals (try to get dust > 0)

- \_convertDecimals(18,2,1e15)
- transferAmount = 0, dust = 1e15
- Line 1092, transferAmount += 1;
- \_convertDecimals(2,18,1)
- normalizedTransferAmount = 1e16, normalizedTruncatedAmount = 0
- Line 1100, normalizedTruncatedAmount is asserted to 0
- normalizedTransferAmount, 1e16 > 1e15.
- dust = 1e16 - 1e15 = 10
- transferAmount = 1, dust = 10
- Issue with low decimals and low shToken mint amount?

5. From 18 decimals to 6 decimals (try to get dust > 0, from the example)

- 0.123456789012345678 to 0.123456789012345678shTokens
- \_convertDecimals(18,6,123456789012345678)
- convertedAmt = 123456, truncatedAmt = 789012345678
- Line 1092, transferAmount = 123457
- \_convertDecimals(6,18,123457)
- normalizedTransferAmt = 123457000000000000, normalizedTruncatedAmt = 0
- dust = 123457000000000000 - 123456789012345678 = 210,987,654,322
- transferAmount = 123457, dust = 210,987,654,322 (0.21e12), amount = 123456789012345678

6. From 18 decimals to 6 decimals (try to get dust > 0, new example)

- 12.345678901234567890 to 12.345678901234567890shTokens (100x more from the last example)
- \_convertDecimals(18,6,12345678901234567890)
- convertedAmt = 12345678, truncatedAmt = 901234567890
- Line 1092, transferAmount = 12345679
- normalizedTransferAmt = 12345679000000000000, normalizedTruncatedAmt = 0
- dust = 12345679000000000000 - 12345678901234567890 = 98,765,432,110
- transferAmount = 12345679, dust = 98,765,432,110, amount = 12345678901234567890
- Comments: less dust than previous example even with 100x more tokens?
- Comments: transferAmount seems right, should be in 6 decimals
- Comments: User deposit 12.3456789 tokens and get 12345678901234567890 shTokens
- Comments: Protocol will get 98,765,432,110 shTokens
- Comments: Total shTokens minted will be 12345678901234567890

7. From 18 decimals to 2 decimals (another example)

- 12.34 tokens to shTokens
- \_convertDecimals(18,2,12345678901234567890)
- convertedAmt = 1234, truncatedAmt = 5678901234567890
- Line 1092, transferAmount = 1235
- normalizedTransferAmt = 12350000000000000000, normalizedTruncatedAmt = 0
- dust = 12350000000000000000 - 12345678901234567890 = 4,321,098,765,432,110
- transferAmount = 1235, dust = 4,321,098,765,432,110, amount = 12345678901234567890
- Comments: Seems like there is no issue with lower decimal tokens
- Comments: transfer 1235 (12.35e2) tokens to get 12345678901234567890 (12.34e18) tokens

(All examples uses Remix. A copy of the code can be found below)

```
pragma solidity 0.8.10;

contract Shell {
    function _convertDecimals(
        uint8 decimalsFrom,
        uint8 decimalsTo,
        uint256 amountToConvert
    ) public returns (uint256 convertedAmount, uint256 truncatedAmount) {
        if (decimalsFrom == decimalsTo) {
            // no shift
            convertedAmount = amountToConvert;
            truncatedAmount = 0;
        } else if (decimalsFrom < decimalsTo) {
            // Decimal shift left (add precision)
            uint256 shift = 10**(uint256(decimalsTo - decimalsFrom));
            convertedAmount = amountToConvert * shift;
            truncatedAmount = 0;
        } else {
            // Decimal shift right (remove precision) -> truncation
            uint256 shift = 10**(uint256(decimalsFrom - decimalsTo));
            convertedAmount = amountToConvert / shift;
            truncatedAmount = amountToConvert % shift;
        }
    }

    function _calculateOceanId(address tokenAddress, uint256 tokenId)
        public
        pure
        returns (uint256)
    {
        return uint256(keccak256(abi.encodePacked(tokenAddress, tokenId)));
    }
}
```

### Mechanism Review

Ocean.sol

1. \_erc20Wrap()

- For \_erc20Wrap(), the second parameter, amount, is already in 18 decimals. The outputToken is in sh format.
- This can be quite confusing because I thought amount means how much tokens I want to wrap, eg if I have 100 USDC, then I should call with amount as 100e6, not 100e18
- Can be more specific in the NATSPEC, state that amount means the 18 decimals version (sh Amount, not token amount)

2. \_erc20Unwrap()

- Usage of payable.transfer may revert due to the 2300 gas limit

3. Computing Output Amount (Swapping from 100 shUSDC to shUSDT)

- `doInteraction()` from Ocean.sol is called
- `_doInteraction()` is executed, calls `_executeInteraction()`
- `_computeOutputAmount()` is called (Line 612)
- `_computeOutputAmount()` is called on the OceanPrimitive (Line 760)
- `unwrapToken()` is called on OceanAdapter.sol, an abstract contract inherited by CurveAdapter (Line 67)
- `doInteraction()` is called on CurveAdapter. **Note that msg.sender has changed when doInteraction() is invoked again**
- Adapter gets 100 USDT, 100shUSDC is burned from Adapter?
- `primitiveOutputAmount()` is called, moving to CuveAdapter
- `wrapToken()` is called, wrapping the USDT back to shUSDT

Fund flow (fees not included):

- 100 shUSDC -> 100 USDC
- 100 USDC -> 100 USDT
- 100 USDT -> 100 shUSDT

- There seems to be two contracts holding funds, one is Ocean.sol and one is the Adapter.sol, which will result in accounting errors.
- For exmaple, The CurveAdapter will not have the shToken to burn when calling unwrap, as the accounting lies in the Ocean contract and user
- Truncation while calculating swaps/deposit via CurveAdapter is ignored.

4. Getting Fees

- Fees is always calculated in the shToken format.
- Fees have a maximum limit and can be zero for small transfers
- Owner can withraw the fees by interacting with the protocol and unwrapping the shTokens

### Potential Attacks Discussion

Q1: Any 1 wei attacks? Eg depositing 1 wei token into Ocean to mess up the ledger
A1: Don't think it can work, no truncation issues or shares being affected by contract balance, only by input and output tokens

Q2: Any vulnerabilities in accepting all types of ERC20?
A2: Mostly user input issues. Two-token addresses is affected in the leger (Low), Fee on transfer tokens will be affected because of accounting error (Medium). Revert on zero-value transfer is a user input issue. Low decimals token may affect the truncation/dust amount when converting decimals (Low). High decimal tokens seem to work fine. ERC777 can be used because its ERC20 backwards compatible, don't seem to have an issue with the potential callbacks from ERC777.

Q3: Any vulnerabilities in accepting ERC721 tokens?
A3: Upgradeable ERC721 tokens will not affect/pausing mechanism will not affect since the Ocean holds the ERC721 token. (ie users wrapping/unwrapping their token will not affect another user). One possible issue is protocol assumes ERC721 tokens from a collection are all the same, but some tokens are more rare than others (Medium).

Q4: Any vulnerabilities in accepting ERC1155 tokens?
A4: Not so much, vulnerabilities will be similar to ERC721.

Q5: Any frontrunning issues?
A5: There's a swap functionality available, but frontrunning is not an issue because of slippage check. Frontrunning is not applicable when wrapping/unwrapping.

Q6: Any gas-related issues?
A6: Usage of payable.transfer when unwrapping ether. Protocol uses safeTransfer, so return value is checked. No other known usage of low-level calls. Gas griefing only affects the user himself.

Q7: Any reentrancy issues?
A7: When unwrapping tokens (burning shToken to Token), there is a point in the function execution whereby the holder holds 2 tokens, the sh variation and the token, which may cause some read-only reentrancy / cross-function reentrancy (Low).

Q8: Can a user grief another person's balance ledger?
A8: \_doInteraction is set to msg.sender, and forwardDoInteraction must be approved, so a user cannot manipulate another user's sh balance.

Q9: Can a user manipulate his own balance?
A9: There's a onERC1155Receive callback when minting shTokens in `_doSafeTransferAcceptanceCheck()` (OceanERC1155.sol, out of scope). User can reenter through the callback, but it doesn't really matter.

### Centralization Issues

- Not much centralization issues. Ledger is extremely decentralized. Owner only has control over the fees, but there is a maximum fee charge




### Time spent:
25 hours