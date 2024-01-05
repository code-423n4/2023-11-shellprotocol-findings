---
sponsor: "Shell Protocol"
slug: "2023-11-shellprotocol"
date: "2024-01-05"
title: "Shell Protocol"
findings: "https://github.com/code-423n4/2023-11-shellprotocol-findings/issues"
contest: 308
---

# Overview

## About C4

Code4rena (C4) is an open organization consisting of security researchers, auditors, developers, and individuals with domain expertise in smart contracts.

A C4 audit is an event in which community participants, referred to as Wardens, review, audit, or analyze smart contract logic in exchange for a bounty provided by sponsoring projects.

During the audit outlined in this document, C4 conducted an analysis of the Shell Protocol smart contract system written in Solidity. The audit took place between November 30 — December 8, 2023.

## Wardens

22 Wardens contributed reports to Shell Protocol:

  1. [peanuts](https://code4rena.com/@peanuts)
  2. [oakcobalt](https://code4rena.com/@oakcobalt)
  3. [EV\_om](https://code4rena.com/@EV_om)
  4. [lsaudit](https://code4rena.com/@lsaudit)
  5. [IllIllI](https://code4rena.com/@IllIllI)
  6. [Udsen](https://code4rena.com/@Udsen)
  7. [0xmystery](https://code4rena.com/@0xmystery)
  8. [bin2chen](https://code4rena.com/@bin2chen)
  9. [Sathish9098](https://code4rena.com/@Sathish9098)
  10. [hunter\_w3b](https://code4rena.com/@hunter_w3b)
  11. [Myd](https://code4rena.com/@Myd)
  12. [0x11singh99](https://code4rena.com/@0x11singh99)
  13. [0xVolcano](https://code4rena.com/@0xVolcano)
  14. [0xepley](https://code4rena.com/@0xepley)
  15. [invitedtea](https://code4rena.com/@invitedtea)
  16. [0xSmartContract](https://code4rena.com/@0xSmartContract)
  17. [LinKenji](https://code4rena.com/@LinKenji)
  18. [albahaca](https://code4rena.com/@albahaca)
  19. [clara](https://code4rena.com/@clara)
  20. [foxb868](https://code4rena.com/@foxb868)
  21. [alexbabits](https://code4rena.com/@alexbabits)
  22. [ZanyBonzy](https://code4rena.com/@ZanyBonzy)

This audit was judged by [0xA5DF](https://code4rena.com/@0xA5DF).

Final report assembled by [thebrittfactor](https://twitter.com/brittfactorC4).

# Summary

The C4 analysis yielded an aggregated total of 0 unique vulnerabilities in the HIGH and MEDIUM severity categories.

Additionally, C4 analysis included 8 reports detailing issues with a risk rating of LOW severity or non-critical. There were also 5 reports recommending gas optimizations.

All of the issues presented here are linked back to their original finding.

# Scope

The code under review can be found within the [C4 Shell Protocol repository](https://github.com/code-423n4/2023-11-shellprotocol), and is composed of 4 smart contracts written in the Solidity programming language and includes 993 lines of Solidity code.

In addition to the known issues identified by the project team, a Code4rena bot race was conducted at the start of the audit. The winning bot, **The_Madaladinator** from warden Madalad, generated the [Automated Findings report](https://gist.github.com/code423n4/640b27a9b9c209b575ed78aa106bd584) and all findings therein were classified as out of scope.

# Severity Criteria

C4 assesses the severity of disclosed vulnerabilities based on three primary risk categories: high, medium, and low/non-critical.

High-level considerations for vulnerabilities span the following key areas when conducting assessments:

- Malicious Input Handling
- Escalation of privileges
- Arithmetic
- Gas use

For more information regarding the severity criteria referenced throughout the submission review process, please refer to the documentation provided on [the C4 website](https://code4rena.com), specifically our section on [Severity Categorization](https://docs.code4rena.com/awarding/judging-criteria/severity-categorization).

# Low Risk and Non-Critical Issues

For this audit, 8 reports were submitted by wardens detailing low risk and non-critical issues. The [report highlighted below](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/236) by **peanuts** received the top score from the judge.

*The following wardens also submitted reports: [EV\_om](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/336), [lsaudit](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/159), [oakcobalt](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/99), [Udsen](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/294), [0xmystery](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/171), [IllIllI](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/151), and [bin2chen](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/52).*

## [01] 2 address tokens will not work properly on the Ocean ERC1155 Ledger.

ERC20 tokens with multiple entry points (known as double entry tokens or two address tokens) such as Synthetix's ProxyERC20 contract will not work properly on the ledger. In the ledger, all tokens are given a specific uint256 value, by calling `_getSpecifiedToken()`, which will call `_calculateOceanId()`.

```
        uint256 specifiedToken = _getSpecifiedToken(interactionType, externalContract, interaction
```

```
   function _calculateOceanId(address tokenAddress, uint256 tokenId) internal pure returns (uint256) {
        return uint256(keccak256(abi.encodePacked(tokenAddress, tokenId)));
    }
```

If a user uses a two-address token, the two address will be considered different, and the Ocean will create 2 separate ledgers instead of 1.

[OceanAdapter.sol#L107](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L107)

## [02] Interactions is susceptible to read-only reentrancy

The interaction flow in Ocean.sol will complete the interaction first before minting/burning the input/output token.

```
        (inputToken, inputAmount, outputToken, outputAmount) = _executeInteraction(
                interaction, interactionType, externalContract, specifiedToken, interaction.specifiedAmount, userAddress
            );
        }

        // if _executeInteraction returned a positive value for inputAmount,
        // this amount must be deducted from the user's Ocean balance
        if (inputAmount > 0) {
            // since uint, same as (inputAmount != 0)
            _burn(userAddress, inputToken, inputAmount);
        }

        // if _executeInteraction returned a positive value for outputAmount,
        // this amount must be credited to the user's Ocean balance
        if (outputAmount > 0) {
            // since uint, same as (outputAmount != 0)
            _mint(userAddress, outputToken, outputAmount);
        }
```

Take the Unwrapping ERC721 interaction as an example. The user have an shERC721 and wants to unwrap it to get his ERC721 token back. When the ERC721 token is transferred to the user, the shERC721 is not burned yet. This means that the user has both shERC721 and ERC721 in the state.

```
    function _erc721Unwrap(address tokenAddress, uint256 tokenId, address userAddress, uint256 oceanId) private {
        IERC721(tokenAddress).safeTransferFrom(address(this), userAddress, tokenId);
        emit Erc721Unwrap(tokenAddress, tokenId, userAddress, oceanId);
    }
```

If the Ocean has more interactions in the future like lending/borrowing, the user can call ERC721Unwrap and do a cross-function reentrancy by reentering `safeTransferFrom()` after getting his ERC721 back but before burning his shERC721. For example, if the lending interaction just needs user to show proof that they have a particular shERC721 on hand, user can do the following:

- WrapERC721 to get shERC721.
- UnwrapERC721.
- After `safeTransferFrom` is called on ERC721, there is a callback to `onERC721Received`.
- User codes the `ERC721Received` to call the lend function.
- Since the lend function only requires the existence of the shERC721, call passes and user gets his loan.
- Callback ends, finishes the execution by burning the shERC721.
- User gets his ERC721 back and his loan.

```
function onERC721Received(
        address _sender, address _from, uint256 _tokenId, bytes memory _data)
        external returns (bytes4 retval) {

        require(msg.sender == address(victim), "Nope");
        // Call the loan function
        return victim.onERC721Received.selector;
}
```

This is hypothetical because there is no loan/borrow function on Ocean. However, the existence of a read-only reentrancy and that the user has 2 tokens at a point in the execution is dangerous. Ideally, burn the `sh(Tokens)` first before interaction. Minting later is fine. At all times, the user should only have 1 token in the state.

[Ocean.sol#L904](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L904)

## [03] User can escape fees on ERC1155 tokens

ERC1155 tokens from external sources may have different decimals, but they are not normalized to 18 decimals. The Ocean ledger takes whatever decimals it is and sets it to the corresponding shToken. For example, if a user wants to wrap an ERC1155 token that has 4 decimals, then they will mint a 4 decimal shToken.

```
function _erc1155Wrap(
        address tokenAddress,
        uint256 tokenId,
        uint256 amount,
        address userAddress,
        uint256 oceanId
    )
        private
    {
        //@audit Note that there is no `_convertDecimals()` call
        if (tokenAddress == address(this)) revert NO_RECURSIVE_WRAPS();
        _ERC1155InteractionStatus = INTERACTION;
        IERC1155(tokenAddress).safeTransferFrom(userAddress, address(this), tokenId, amount, "");
        _ERC1155InteractionStatus = NOT_INTERACTION;
        emit Erc1155Wrap(tokenAddress, tokenId, amount, userAddress, oceanId);
    }
```

The fee is therefore dependent on the amount of decimals that the ERC1155 token has. If a user wraps 100 tokens with 2 decimal places, eg `100e2`, they can choose not to pay the fees by calling `erc1155Unwrap` 5 times instead of unwrapping `100e2` tokens directly. This is because the `unwrapFeeDivisor` has a `minValue` of 2000, so if the user can make the `unwrapAmount` less than 2000, then they will not need to pay the fees. (Instead of unwrapping 10000 tokens and paying 5 tokens as fee, user unwraps 2000 tokens 5 times).

```
    function _calculateUnwrapFee(uint256 unwrapAmount) private view returns (uint256 feeCharged) {
        feeCharged = unwrapAmount / unwrapFeeDivisor;
    }
```

## [04] Airdrops will be locked in the contract

There are some collection of ERC721 tokens that will airdrop users tokens if they have proof that they own an ERC721. There are other ERC721 collection that will airdrop tokens to every holder of the token. If the user deposits the ERC721 token which is eligible for an airdrop into the Ocean protocol, and an airdrop happens, the Ocean will receive the airdrop instead.

Since the ocean contract does not have any way to withdraw random tokens, the airdrop will be locked inside the protocol.

Not sure about the recommendation, but take note that some ERC721 collections may mint tokens for every holder or may airdrop tokens to holders once in a while. Make sure that these tokens are either not accepted into Ocean, or that the users are warned beforehand that the airdrop will be lost.

[Ocean.sol#L79](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L79)

## [05] NATSPEC on `OceanAdapter._convertDecimals` should be revised

The function `_convertDecimals()` from OceanAdapter.sol is an exact copy on Ocean.sol. However, the OceanAdapter version does not return the `truncatedAmount`.

```
(OceanAdapter.sol)
function _convertDecimals(
        uint8 decimalsFrom,
        uint8 decimalsTo,
        uint256 amountToConvert
    )
        internal
        pure
>       returns (uint256 convertedAmount)
```

```
 function _convertDecimals(
        uint8 decimalsFrom,
        uint8 decimalsTo,
        uint256 amountToConvert
    )
        internal
        pure
>       returns (uint256 convertedAmount, uint256 truncatedAmount)
```

The comments on OceanAdapter.sol should be revised accordingly to not reference the `truncatedAmount`.

```
     * @dev convert a uint256 from one fixed point decimal basis to another,
     *   returning the truncated amount if a truncation occurs.
```

[OceanAdapter.sol#L129-L130](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L129-L130)

## [06] `minimumOutputAmount` in CurveAdaptor can be integrated into the curve functionalities

The Curve Adapters interact with curve pool through three functions, `exchange()`, `remove_liquidity_one_coin()` and `add_liquidity()`. The last parameter of these functions are set to zero, which is the slippage parameter.

```
 ICurveTricrypto(primitive).remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);
```

Ocean protocol takes care of it by checking the slippage right at the end, but it can also be integrated right into the curve functions.

```
        if (uint256(minimumOutputAmount) > outputAmount) revert SLIPPAGE_LIMIT_EXCEEDED();
```

[CurveTricryptoAdapter.sol#L201-L203](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L201-L203)

## [07] Comments on ERC20Wrap are misleading

In ERC20Wrap, the `amount` parameter is supposed to be the amount of shTokens to receive, not the amount of ERC20 tokens to be wrapped.

```
>    * @param amount amount of the ERC-20 token to be wrapped, in terms of
     *  18-decimal fixed point
     * @param userAddress the address of the user who is wrapping the token
     */
>   function _erc20Wrap(address tokenAddress, uint256 amount, address userAddress, uint256 outputToken) private {
```

The `outputAmount` is the `specifiedAmount`, which is passed as `amount` into ERC20Wrap function.

```
        } else if (interactionType == InteractionType.WrapErc20) {
            inputToken = 0;
            inputAmount = 0;
            outputToken = specifiedToken;
>          outputAmount = specifiedAmount;
>           _erc20Wrap(externalContract, outputAmount, userAddress, outputToken);
```

This `outputAmount` is then minted as the shToken, which means that the `outputAmount` must be in the form of shTokens.

```
        if (outputAmount > 0) {
            // since uint, same as (outputAmount != 0)
            _mint(userAddress, outputToken, outputAmount);
        }
```

It will be useful if `amount` is more specific and better explained in the comments. Is `amount` referring to the token amount to deposit or the corresponding shToken to receive?

[Ocean.sol#L426-L431](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L426-L431)

**[0xA5DF (judge) commented](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/236#issuecomment-1858752730):**
> [01] - Low<br>
> [02] - Low<br>
> [03] - Non-Critical<br>
> [04] - Non-Critical<br>
> [05] - Low<br>
> [06] - Refactor<br>
> [07] - Non-Critical

## [[08] Extreme edge case error results in user paying more fees than usual when user decides to mint little shTokens with low decimals tokens](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/272)

*Note: At the judge's request [here](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/236#issuecomment-1858762123), this downgraded issue from the same warden has been included in this report for completeness.*

### Lines of code

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L1068-L1109

### Proof of Concept

The wrapping and unwrapping process is mostly foolproof even with low/high decimal tokens. There is an edge case if a person intends to mint `~1e12` shTokens with a low decimal token.

Let's say that token ABC has 2 decimals and is worth 20000 USDC (`1e2 ABC tokens == 20000`).

User wants to mint `1e15` shABC tokens

- `convertDecimals` (`18,2,1e15`).
- `convertedAmount` `= 0`, `truncatedAmount` `= 1000000000000000`.
- Since `truncatedAmount` `> 0`, Line 1084 is executed.
- `transferAmount` `= 1`.
- `converDecimals` (`2,18,1`).
- `normalizedTransferAmount` `= 10000000000000000`, `normalizedTruncatedAmount` `= 0`.
- `dust =` `normalizedTransferAmount` `- 1e15 = 9e15`.
- `transferAmount` `= 1`, `dust = 9e15`.

In order to mint `1e15` shTokens, user has to pay 1 wei token to get `1e15` tokens. `9e15` shTokens is paid to the the protocol. By right, 1 wei token should get the user `1e16` shTokens instead of `1e15` shTokens.

Code Reference: 

```
    function _determineTransferAmount(
        uint256 amount,
        uint8 decimals
    )
        private
        pure
        returns (uint256 transferAmount, uint256 dust)
    {
        // if (decimals < 18), then converting 18-decimal amount to decimals
        // transferAmount will likely result in amount being truncated. This
        // case is most likely to occur when a user is wrapping a delta as the
        // final interaction in a transaction.
        uint256 truncated;

        (transferAmount, truncated) = _convertDecimals(NORMALIZED_DECIMALS, decimals, amount);

        if (truncated > 0) {
            // Here, FLOORish(x) is not to the nearest integer less than `x`,
            // but rather to the nearest value with `decimals` precision less
            // than `x`. Likewise with CEILish(x).
            // When truncating, transferAmount is FLOORish(amount), but to
            // fully cover a potential delta, we need to transfer CEILish(amount)
            // if truncated == 0, FLOORish(amount) == CEILish(amount)
            // When truncated > 0, FLOORish(amount) + 1 == CEILish(AMOUNT)
            transferAmount += 1;
            // Now that we are transferring enough to cover the delta, we
            // need to determine how much of the token the user is actually
            // wrapping, in terms of 18-decimals.
            (uint256 normalizedTransferAmount, uint256 normalizedTruncatedAmount) =
                _convertDecimals(decimals, NORMALIZED_DECIMALS, transferAmount);
            // If we truncated earlier, converting the other direction is adding
            // precision, which cannot truncate.
            assert(normalizedTruncatedAmount == 0);
            assert(normalizedTransferAmount > amount);
            dust = normalizedTransferAmount - amount;
        } else {
            // if truncated == 0, then we don't need to do anything fancy to
            // determine transferAmount, the result _convertDecimals() returns
            // is correct.
            dust = 0;
        }
    }
```

### Impact

User pays more fees than normal when wrapping tokens with low decimals and minting low amounts of shTokens.

### Tools Used

VSCode

### Recommended Mitigation Steps

Recommend not allowing minting such a low amount of shTokens and also possibly disallowing extremely low decimal tokens.

### Assessed type

Error

**[0xA5DF (judge) decreased severity to QA](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/272#issuecomment-1857724011)**

## [[09] Owner has to perpetually pay fees to withdraw fees, resulting in dust amount left in the contract](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/221)

*Note: At the judge's request [here](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/236#issuecomment-1858762123), this downgraded issue from the same warden has been included in this report for completeness.*

### Lines of code

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L1166-L1171

### Proof of Concept

The owner gets fees every time `_grantFeeToOcean()` is invoked, (from unwrapping ether, ERC1155, ERC20 and wrapping ERC20).

```
  function _grantFeeToOcean(uint256 oceanId, uint256 amount) private {
        if (amount > 0) {
            // since uint, same as (amount != 0)
            _mintWithoutSafeTransferAcceptanceCheck(owner(), oceanId, amount);
        }
    }
```

This fee is usually in the shToken format (except for the ERC1155 fees, which follows the ERC1155 token decimals), so when the owner wants to withdraw his fees in the ERC20 format, he will have to call unwrapERC20 as well.

```
    function _erc20Unwrap(address tokenAddress, uint256 amount, address userAddress, uint256 inputToken) private {
        try IERC20Metadata(tokenAddress).decimals() returns (uint8 decimals) {
>           uint256 feeCharged = _calculateUnwrapFee(amount);
            uint256 amountRemaining = amount - feeCharged;

            (uint256 transferAmount, uint256 truncated) =
                _convertDecimals(NORMALIZED_DECIMALS, decimals, amountRemaining);
            feeCharged += truncated;

>           _grantFeeToOcean(inputToken, feeCharged);

            SafeERC20.safeTransfer(IERC20(tokenAddress), userAddress, transferAmount);
            emit Erc20Unwrap(tokenAddress, transferAmount, amount, feeCharged, userAddress, inputToken);
        } catch {
            revert NO_DECIMAL_METHOD();
        }
    }
```

When the owner calls the ERC20Unwrap interaction, he will have to pay fees as well. This means that whenever he wants to withdraw a sum of tokens, he will have to pay fees. Then, when he wants to withdraw the fees he paid, he will have to pay fees again. This will lead to a small sum of fees left inside the contract that cannot be withdrawn.

Eg. Owner has about `100e18` of shUSDC accumulated from fees that he wants to withdraw. 

- He calls `_erc20Unwrap()` to unwrap `100e18` shUSDC.
- Assuming `unwrapFeeDivisor` is 2000, the fees he has to pay is `100e18 / 2000 = 5e16`.
- He gets `99.95e6` USDC back but `0.05e6` USDC is left in the contract.
- He calls `_erc20Unwrap()` to unwrap `5e16` shUSDC.
- He has to pay `5e16 / 2000` fees which is `2.5e13`.
- He gets a small sum of USDC back but another smaller sum of USDC is left in the contract.

This goes on perpetually; the owner cannot withdraw all his fees entirely.

### Impact

Owner will have some fees stuck in the contract.

### Tools Used

VSCode

### Recommended Mitigation Steps

When calling `_grantFeeToOcean()`, if the `msg.sender` is the owner, then skip the function so that the owner doesn't have to pay his own fees.

```
  function _grantFeeToOcean(uint256 oceanId, uint256 amount) private {
>       if (amount > 0 && msg.sender != owner()) {
            // since uint, same as (amount != 0)
            _mintWithoutSafeTransferAcceptanceCheck(owner(), oceanId, amount);
        }
    }
```

### Assessed type

Invalid Validation

**[0xA5DF (judge) decreased severity to QA](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/221#issuecomment-1857368362)**

***

# Gas Optimizations

For this audit, 5 reports were submitted by wardens detailing gas optimizations. The [report highlighted below](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/274) by **hunter_w3b** received the top score from the judge.

*The following wardens also submitted reports: [0x11singh99](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/224), [0xVolcano](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/220), [IllIllI](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/154), and [Sathish9098](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/130).*

## [G-01] Avoid contract existence checks by using low level calls

**Issue Description** - Prior to Solidity `0.8.10`, the compiler would insert extra code to check for contract existence before external function calls,
even if the call had a return value. This wasted gas by performing a check that wasn't necessary.

**Proposed Optimization** - For functions that make external calls with return values in Solidity `<0.8.10`, optimize the code to use low-level calls instead
of regular calls. Low-level calls skip the unnecessary contract existence check.

Example:

```solidity
//Before:

contract C {
  function f() external returns(uint) {
    address(otherContract).call(abi.encodeWithSignature("func()"));
  }
}


//After:

contract C {
  function f() external returns(uint) {
    (bool success,) = address(otherContract).call(abi.encodeWithSignature("func()"));
    require(success);
    return decodeReturnValue();
  }
}
```

**Estimated Gas Savings** - Each avoided `EXTCODESIZE` check saves 100 gas. If 10 external calls are made in a common function, this would save 1000 gas
total.

**Code Snippets:**

```solidity
File: adapters/Curve2PoolAdapter.sol

78        address xTokenAddress = ICurve2Pool(primitive).coins(0);

81        decimals[xToken] = IERC20Metadata(xTokenAddress).decimals();

84        address yTokenAddress = ICurve2Pool(primitive).coins(1);

88        decimals[yToken] = IERC20Metadata(yTokenAddress).decimals();

93        decimals[lpTokenId] = IERC20Metadata(primitive_).decimals();

113        IOceanInteractions(ocean).doInteraction(interaction);

132        IOceanInteractions(ocean).doInteraction(interaction);

164                ICurve2Pool(primitive).exchange(indexOfInputAmount, indexOfOutputAmount, rawInputAmount, 0);

168            rawOutputAmount = ICurve2Pool(primitive).add_liquidity(inputAmounts, 0);

170            rawOutputAmount = ICurve2Pool(primitive).remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);

190        IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);

191        IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L78

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

86        address xTokenAddress = ICurveTricrypto(primitive).coins(0);

89        decimals[xToken] = IERC20Metadata(xTokenAddress).decimals();

92        address yTokenAddress = ICurveTricrypto(primitive).coins(1);

96        decimals[yToken] = IERC20Metadata(yTokenAddress).decimals();

99        address wethAddress = ICurveTricrypto(primitive).coins(2);

106        address lpTokenAddress = ICurveTricrypto(primitive).token();

109        decimals[lpTokenId] = IERC20Metadata(lpTokenAddress).decimals();

129            IOceanInteractions(ocean).doInteraction{ value: amount }(interaction);

138            IOceanInteractions(ocean).doInteraction(interaction);

168        IOceanInteractions(ocean).doInteraction(interaction);

210            ICurveTricrypto(primitive).add_liquidity(inputAmounts, 0);

213                uint256 wethBalance = IERC20Metadata(underlying[zToken]).balanceOf(address(this));

214                ICurveTricrypto(primitive).remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);

215                IWETH(underlying[zToken]).withdraw(

219                ICurveTricrypto(primitive).remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);

242        IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);

243        IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L86

```solidity
File: src/ocean/Ocean.sol

891        IERC721(tokenAddress).safeTransferFrom(userAddress, address(this), tokenId);

904        IERC721(tokenAddress).safeTransferFrom(address(this), userAddress, tokenId);

931        IERC1155(tokenAddress).safeTransferFrom(userAddress, address(this), tokenId, amount, "");

968        IERC1155(tokenAddress).safeTransferFrom(address(this), userAddress, tokenId, amountRemaining, "");

```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L891

## [G-02] Add unchecked `{}` for subtractions where the operands can't underflow because of previous checks

**Issue Description** - The subtraction operation `dust = normalizedTransferAmount - amount`; does not have an unchecked block, even though underflow
is not possible due to the previous check `normalizedTransferAmount > amount`. Adding an unchecked block helps avoid unnecessary
checks.

**Proposed Optimization** - Add an unchecked block around the subtraction:

```solidity
unchecked {
  dust = normalizedTransferAmount - amount;
}
```

**Estimated Gas Savings** - Adding an unchecked block for a subtraction that cannot underflow due to a previous check saves approximately 3 gas per subtraction.
With many such subtractions occurring across interactions, the total gas savings could be significant.

**Code Snippets:**

```solidity
File: src/ocean/Ocean.sol

1102            dust = normalizedTransferAmount - amount;
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L1102

```solidity
File: src/adapters/OceanAdapter.sol

152            uint256 shift = 10 ** (uint256(decimalsTo - decimalsFrom));
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L152

## [G-03] State variables should be cached in stack variables rather than re-reading them from storage

**Issue Description** - In the `changeUnwrapFee()` function, the state variable `unwrapFeeDivisor` is read directly from storage on line 199 when emitting the event `ChangeUnwrapFee`. This is inefficient, as it incurs an extra storage read operation that is unnecessary.

**Proposed Optimization** - Cache the current value of `unwrapFeeDivisor` in a local stack variable before emitting the event and updating storage. This
avoids an unnecessary re-read of the storage.

```solidity
function changeUnwrapFee(uint256 nextUnwrapFeeDivisor) external override onlyOwner {
  uint256 currentUnwrapFeeDivisor = unwrapFeeDivisor;

  if (MIN_UNWRAP_FEE_DIVISOR > nextUnwrapFeeDivisor) revert();

  emit ChangeUnwrapFee(currentUnwrapFeeDivisor, nextUnwrapFeeDivisor, msg.sender);

  unwrapFeeDivisor = nextUnwrapFeeDivisor;
}
```

**Estimated Gas Savings** - Reading a storage slot costs roughly 200 gas. Caching the value in a local variable avoids this extra read, saving roughly
200 gas per function call.

**Code Snippet:**

```solidity
File: src/ocean/Ocean.sol

196    function changeUnwrapFee(uint256 nextUnwrapFeeDivisor) external override onlyOwner {
197        /// @notice as the divisor gets smaller, the fee charged gets larger
198        if (MIN_UNWRAP_FEE_DIVISOR > nextUnwrapFeeDivisor) revert();
199        emit ChangeUnwrapFee(unwrapFeeDivisor, nextUnwrapFeeDivisor, msg.sender); //@audit >>> unwrapFeeDivisor is state-variable
200        unwrapFeeDivisor = nextUnwrapFeeDivisor;
201    }



1082        (transferAmount, truncated) = _convertDecimals(NORMALIZED_DECIMALS, decimals, amount); //@audit >>>
1097                _convertDecimals(decimals, NORMALIZED_DECIMALS, transferAmount);        //@audit >>>
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L196-L201

## [G-04] Using assembly to revert with an error message

**Issue Description** - When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error
message. This can in most cases be further optimized by using assembly to revert with the error message.

**Estimated Gas Savings** - Here’s an example:

```solidity
/// calling restrictedAction(2) with a non-owner address: 24042


contract SolidityRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num)  external {
        require(owner == msg.sender, "caller is not owner");
        specialNumber = num;
    }
}

/// calling restrictedAction(2) with a non-owner address: 23734


contract AssemblyRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num)  external {
        assembly {
            if sub(caller(), sload(owner.slot)) {
                mstore(0x00, 0x20) // store offset to where length of revert message is stored
                mstore(0x20, 0x13) // store length (19)
                mstore(0x40, 0x63616c6c6572206973206e6f74206f776e657200000000000000000000000000) // store hex representation of message
                revert(0x00, 0x60) // revert with data
            }
        }
        specialNumber = num;
    }
}
```

From the example above, we can see that we get a gas saving of over 300 gas when reverting with the same error message with assembly against doing so in solidity. This gas savings come from the memory expansion costs and extra type checks the solidity compiler does under the hood.

**Code Snippets:**

```solidity
File: src/ocean/Ocean.sol

640            if (specifiedAmount != 1) revert INVALID_ERC721_AMOUNT();

648            if (specifiedAmount != 1) revert INVALID_ERC721_AMOUNT();

840            revert NO_DECIMAL_METHOD();

878            revert NO_DECIMAL_METHOD();

929        if (tokenAddress == address(this)) revert NO_RECURSIVE_WRAPS();

964        if (tokenAddress == address(this)) revert NO_RECURSIVE_UNWRAPS();
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L640

```solidity
File: src/adapters/Curve2PoolAdapter.sol

175        if (uint256(minimumOutputAmount) > outputAmount) revert SLIPPAGE_LIMIT_EXCEEDED();

217            revert INVALID_COMPUTE_TYPE();
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L175

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

227        if (uint256(minimumOutputAmount) > outputAmount) revert SLIPPAGE_LIMIT_EXCEEDED();

287            revert INVALID_COMPUTE_TYPE();
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L227

## [G-05] Use assembly to reuse memory space when making more than one external call

**Issue Description** - When making external calls, the solidity compiler has to encode the function signature and arguments in memory. It does not
clear or reuse memory, so it expands memory each time.

**Proposed Optimization** - Use inline assembly to reuse the same memory space for multiple external calls. Store the function selector and arguments
without expanding memory further.

**Estimated Gas Savings** - Reusing memory can save thousands of gas compared to expanding on each call. The baseline memory expansion per call is 3,000
gas. With larger arguments or return data, the savings would be even greater.

```solidity
See the example below;

contract Called {
    function add(uint256 a, uint256 b) external pure returns(uint256) {
        return a + b;
    }
}


contract Solidity {
    // cost: 7262
    function call(address calledAddress) external pure returns(uint256) {
        Called called = Called(calledAddress);
        uint256 res1 = called.add(1, 2);
        uint256 res2 = called.add(3, 4);

        uint256 res = res1 + res2;
        return res;
    }
}


contract Assembly {
    // cost: 5281
    function call(address calledAddress) external view returns(uint256) {
        assembly {
            // check that calledAddress has code deployed to it
            if iszero(extcodesize(calledAddress)) {
                revert(0x00, 0x00)
            }

            // first call
            mstore(0x00, hex"771602f7")
            mstore(0x04, 0x01)
            mstore(0x24, 0x02)
            let success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res1 := mload(0x60)

            // second call
            mstore(0x04, 0x03)
            mstore(0x24, 0x4)
            success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res2 := mload(0x60)

            // add results
            let res := add(res1, res2)

            // return data
            mstore(0x60, res)
            return(0x60, 0x20)
        }
    }
}
```

We save approximately 2,000 gas by using the scratch space to store the function selector and its arguments and also reusing the same memory space for the second call while storing the returned data in the zero slot thus not expanding memory.

If the arguments of the external function you wish to call is above 64 bytes and if you are making one external call, it wouldn’t save any significant gas writing it in assembly. However, if making more than one call. You can still save gas by reusing the same memory slot for the 2 calls using inline assembly.

*Note: Always remember to update the free memory pointer if the offset it points to is already used, to avoid solidity overriding the data stored there or using the value stored there in an unexpected way.*

Also note to avoid overwriting the zero slot (`0x60` memory offset) if you have undefined dynamic memory values within that call stack. An alternative is to explicitly define dynamic memory values or if used, to set the slot back to `0x00` before exiting the assembly block.

**Code Snippets:**

```solidity
File: src/adapters/Curve2PoolAdapter.sol

78        address xTokenAddress = ICurve2Pool(primitive).coins(0);
81        decimals[xToken] = IERC20Metadata(xTokenAddress).decimals();
84        address yTokenAddress = ICurve2Pool(primitive).coins(1);
88        decimals[yToken] = IERC20Metadata(yTokenAddress).decimals();
93        decimals[lpTokenId] = IERC20Metadata(primitive_).decimals();



164                ICurve2Pool(primitive).exchange(indexOfInputAmount, indexOfOutputAmount, rawInputAmount, 0);
168            rawOutputAmount = ICurve2Pool(primitive).add_liquidity(inputAmounts, 0);
170            rawOutputAmount = ICurve2Pool(primitive).remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);



190        IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);
191        IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L78

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

86        address xTokenAddress = ICurveTricrypto(primitive).coins(0);
89        decimals[xToken] = IERC20Metadata(xTokenAddress).decimals();
92        address yTokenAddress = ICurveTricrypto(primitive).coins(1);
96        decimals[yToken] = IERC20Metadata(yTokenAddress).decimals();
99        address wethAddress = ICurveTricrypto(primitive).coins(2);
106        address lpTokenAddress = ICurveTricrypto(primitive).token();
109        decimals[lpTokenId] = IERC20Metadata(lpTokenAddress).decimals();



129            IOceanInteractions(ocean).doInteraction{ value: amount }(interaction);
138            IOceanInteractions(ocean).doInteraction(interaction);


201            ICurveTricrypto(primitive).exchange{ value: inputToken == zToken ? rawInputAmount : 0 }(
208            if (inputToken == zToken) IWETH(underlying[zToken]).deposit{ value: rawInputAmount }();
210            ICurveTricrypto(primitive).add_liquidity(inputAmounts, 0);
213                uint256 wethBalance = IERC20Metadata(underlying[zToken]).balanceOf(address(this));
214                ICurveTricrypto(primitive).remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);
215                IWETH(underlying[zToken]).withdraw(
216                    IERC20Metadata(underlying[zToken]).balanceOf(address(this)) - wethBalance
219                ICurveTricrypto(primitive).remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);


242       IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);
243        IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L86

This report discusses how using inline assembly can optimize gas costs when making multiple external calls by reusing memory space, rather than expanding memory separately for each call. This can save thousands of gas compared to the
solidity compiler's default behavior.

## [G-06] Don't make variables public unless necessary

**Issue Description** - Making variables public comes with some overhead costs that can be avoided in many cases. A public variable implicitly creates a public getter function of the same name, increasing the contract size.

**Proposed Optimization** - Only mark variables as public if their values truly need to be readable by external contracts/users. Otherwise, use private
or internal visibility.

**Estimated Gas Savings** - The savings from avoiding unnecessary public variables are small per transaction, around 5-10 gas. However, this adds up
over many transactions targeting a contract with public state variables that don't need to be public.

**Code Snippets:**

```solidity
File: src/ocean/Ocean.sol

84    uint256 public immutable WRAPPED_ETHER_ID;

90    uint256 public unwrapFeeDivisor;
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L84

```solidity
File: src/adapters/Curve2PoolAdapter.sol

56    uint256 public immutable xToken;

59    uint256 public immutable yToken;

62    uint256 public immutable lpTokenId;
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L56

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

61    uint256 public immutable xToken;

64    uint256 public immutable yToken;

67    uint256 public immutable zToken;

70    uint256 public immutable lpTokenId;
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L61

## [G-07] It is sometimes cheaper to cache calldata

**Issue Description** - While the `calldataload` opcode is relatively cheap, directly using it in a loop or multiple times can still result in unnecessary
bytecode. Caching the loaded calldata first may allow for optimization opportunities.

**Proposed Optimization** - Cache calldata values in a local variable after first load, then reference the local variable instead of repeatedly using
calldataload.

**Estimated Gas Savings** - Exact savings vary depending on contract, but caching calldata parameters can save 5-20 gas per usage by avoiding extra `calldataload` opcodes. Larger functions with many parameter uses see more benefit.

**Code Snippet:**

```solidity
File: src/ocean/Ocean.sol

230        Interaction[] calldata interactions,
231        uint256[] calldata ids


282        Interaction[] calldata interactions,
283        uint256[] calldata ids,

356        uint256[] calldata,
357        uint256[] calldata,

446        Interaction[] calldata interactions,
447        uint256[] calldata ids,
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L230-L231

## [G-08] Shorten arrays with inline assembly

**Issue Description** - When shortening an array in Solidity, it creates a new shorter array and copies the elements over. This wastes gas by duplicating
storage.

**Proposed Optimization** - Use inline assembly to shorten the array in place by changing its length slot, avoiding the need to copy elements to a new
array.

**Estimated Gas Savings** - Shortening a `length-n` array avoids `~n` SSTORE operations to copy elements. Benchmarking shows savings of 5000-15000 gas depending on original length.

```solidity
function shorten(uint[] storage array, uint newLen) internal {

  assembly {
    sstore(array_slot, newLen)
  }

}

// Rather than:
function shorten(uint[] storage array, uint newLen) internal {

  uint[] memory newArray = new uint[](newLen);

  for(uint i = 0; i < newLen; i++) {
    newArray[i] = array[i];
  }

  delete array;
  array = newArray;

}
```

Using inline assembly allows shortening arrays without copying elements to a new storage slot, providing significant gas savings.

**Code Snippet:**

```solidity
File: src/ocean/Ocean.sol

460        BalanceDelta[] memory balanceDeltas = new BalanceDelta[](ids.length);
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L460

## [G-09] Use `bytes.concat()` instead of `abi.encodePacked()`, since this is preferred since `0.8.4`

**Issue Description** - The `abi.encodePacked()` function is used to concatenate multiple arguments into a byte array prior to `0.8.4`. However, since `0.8.4` the `bytes.concat()` function was introduced which performs the same role but is preferred since it avoids ABI encoding overhead.

**Proposed Optimization** - Replace uses of `abi.encodePacked()` with `bytes.concat()` where multiple arguments need to be concatenated into a byte array.

**Estimated Gas Savings** - Using `bytes.concat()` instead of `abi.encodePacked()` saves approximately 300-500 gas per concatenation by avoiding ABI encoding.

**Code Snippet:**

```solidity
File: src/adapters/OceanAdapter.sol

109        return uint256(keccak256(abi.encodePacked(tokenAddress, tokenId)));
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L109

## [G-10] Private functions used once can be inlined

**Issue Description** - Private functions that are only used internally in a single caller do not provide any abstraction benefits. The function call overhead is wasted gas in this case.

**Proposed Optimization** - Identify private functions that are only used in a single internal caller, and inline their code directly into the caller
by copy-pasting the function body. This avoids the unnecessary function call.

**Estimated Gas Savings** - Inlining a private function saves approximately 200-500 gas by removing the function call overhead. This adds up over many
private functions.

**Code Snippets:**

```solidity
File: src/ocean/Ocean.sol

820    function _erc20Wrap(address tokenAddress, uint256 amount, address userAddress, uint256 outputToken) private {

864    function _erc20Unwrap(address tokenAddress, uint256 amount, address userAddress, uint256 inputToken) private {

889    function _erc721Wrap(address tokenAddress, uint256 tokenId, address userAddress, uint256 oceanId) private {

903    function _erc721Unwrap(address tokenAddress, uint256 tokenId, address userAddress, uint256 oceanId) private {

920    function _erc1155Wrap(

955    function _erc1155Unwrap(

978    function _etherUnwrap(uint256 amount, address userAddress) private {

1068    function _determineTransferAmount(
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L1068

```solidity
File: src/adapters/Curve2PoolAdapter.sol

201    function _determineComputeType(
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L201

```solidity
File: src/adapters/CurveTricryptoAdapter.sol

264    function _determineComputeType(
```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L264

**[0xA5DF (judge) commented](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/274#issuecomment-1858829009):**
> [G-05] - the savings seem to actually be ~450 per instance (when optimization is turned on), and I'm not sure it'd work for every case there.<br>
> [G-09] -  in bot findings.


*Note: For full discussion, see [here](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/274).*

***

# Audit Analysis

For this audit, 13 analysis reports were submitted by wardens. An analysis report examines the codebase as a whole, providing observations and advice on such topics as architecture, mechanism, or approach. The [report highlighted below](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/304) by **Sathish9098** received the top score from the judge.

*The following wardens also submitted reports: [Myd](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/219), [0xepley](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/332), [invitedtea](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/326), [0xSmartContract](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/311), [peanuts](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/267), [LinKenji](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/251), [albahaca](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/240), [clara](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/227), [foxb868](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/223), [oakcobalt](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/145), [alexbabits](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/102), and [ZanyBonzy](https://github.com/code-423n4/2023-11-shellprotocol-findings/issues/28).*

## Overview 

Shell Protocol is a decentralized finance (DeFi) platform designed to simplify and enhance the DeFi experience for both users and developers

**One-Click Transactions** - Shell Protocol aims to streamline the user experience by offering powerful, one-click transactions. This feature is particularly beneficial for those new to DeFi or those seeking a more efficient and less complex interaction with DeFi services.

**Capital-Efficient Automated Market Makers (AMMs)** - At the heart of Shell Protocol is its set of Automated Market Makers, which are touted for their exceptional capital efficiency. This means that liquidity providers can expect better returns on their assets, and traders can enjoy lower slippage and more stable prices. The AMMs are designed to be robust and flexible, catering to a wide range of financial assets.

**Modular Developer Experience** - Recognizing the diverse needs of DeFi developers, Shell Protocol offers a modular design. This approach allows developers to easily integrate Shell's features into their projects or to build on top of the platform. The modular design also facilitates customization and scalability, enabling developers to create tailored DeFi solutions.

## Project’s risk model

### Admin abuse risks

```solidity
196:     function changeUnwrapFee(uint256 nextUnwrapFeeDivisor) external override onlyOwner {
```

### `onlyOwner` (Function: `changeUnwrapFee()`):
This modifier restricts the execution of the function to the owner of the contract.

**Risk** - If the owner account is compromised, the attacker could manipulate the unwrap fee, potentially causing financial loss to users or destabilizing the protocol. Additionally, even without malicious intent, the owner has significant power to alter contract parameters, which can lead to a lack of trust if changes are made arbitrarily or without community consensus.

```solidity
289:         onlyApprovedForwarder(userAddress)
```

### `onlyApprovedForwarder(userAddress)` (Functions: `forwardedDoInteraction()` and `forwardedDoMultipleInteractions()`):
This modifier likely allows only an approved forwarder address, linked to the `userAddress`, to execute the function.

**Risk** - The implementation details of how a forwarder is approved are crucial. If the process of approving forwarders is not decentralized or transparent, it can lead to centralization risks where the administrator could favor certain forwarders. Also, if the mechanism for approval is not secure, it might be susceptible to manipulation, allowing unauthorized entities to act as forwarders.

```solidity
64:         onlyOcean
```

### `onlyOcean` (Functions: `computeOutputAmount()` and `computeInputAmount()`):

This modifier restricts function execution to addresses authorized by the Ocean protocol.

**Risk** - Similar to `onlyOwner`, this creates a single point of failure. If the Ocean authority is compromised or acts maliciously, they could potentially manipulate these functions for personal gain or disrupt the protocol's operations. This risk is compounded if the Ocean authority has a broad range of powers over many aspects of the protocol.

### Mitigation Strategies

- **Decentralized Governance** - Transitioning from a single owner model to a decentralized governance structure can mitigate risks associated with the `onlyOwner` modifier. This would involve community members or token holders voting on critical decisions, thus distributing power.
- **Transparent Approval Process** - For the `onlyApprovedForwarder` modifier, ensuring a transparent and secure process for approving forwarders is essential. This might include community oversight or stringent security checks.
- **Role Rotation and Multi-Signature Controls** - Implementing multi-signature wallets and regularly rotating the roles responsible for critical functions can reduce risks associated with the `onlyOcean` modifier. This ensures that no single entity has prolonged, unchecked control.

## Systemic risks

**Token Wrapping and Unwrapping Risk in Fee Management** - The contract manages fees for wrapping and unwrapping tokens, particularly with ERC-20 and ERC-1155 tokens. Any miscalculation or manipulation in fee management can lead to economic vulnerabilities, affecting user incentives and the protocol's sustainability.

**Risks in Forwarded Interactions** - The `forwardedDoInteraction` and `forwardedDoMultipleInteractions` functions, which use `onlyApprovedForwarder`, introduce additional risks. If the logic for approving forwarders is flawed or if a forwarder is compromised, it could lead to unauthorized or malicious transactions.

**Risk of Slippage and Price Impact** - The contract checks for slippage limits (`SLIPPAGE_LIMIT_EXCEEDED`). If the actual output amount of a swap, deposit, or withdrawal is lower than the user's expected minimum, the transaction is reverted. This mechanism can protect users from high slippage, but if not calibrated correctly, it could also lead to failed transactions during high volatility, impacting the user experience and trust in the protocol.

**ERC20 Token Approval Risks** - The contract grants maximum (`type(uint256).max`) approval to the ocean and primitive addresses for ERC20 tokens. While this is a common pattern to reduce transaction costs, it also means that if either of these addresses is compromised, it could lead to the loss of user funds.

**Conversion and Rounding Errors** - The contract uses `_convertDecimals` for normalizing token amounts to the Ocean's 18-decimal standard. Inaccuracies or rounding errors in these conversions could lead to financial discrepancies, especially for tokens with different decimal standards.

## Technical Risks

**Complex Token Wrapping/Unwrapping Logic** - The logic for wrapping and unwrapping tokens is intricate, with multiple points of interaction with the contract's state and external tokens. Bugs or flaws here could lead to incorrect token balances or loss of funds.

**Non-upgradeability Concerns** - The contract's apparent lack of upgradeability means that addressing any discovered vulnerabilities or design issues could be challenging, posing long-term systemic risks in a rapidly evolving DeFi landscape.

**Gas Usage and Network Load** - The complex operations of the contract may lead to high gas costs, affecting its usability and performance, especially during times of network congestion.

**Complexity in Handling Multiple Token Standards** - The contract interacts with multiple token standards, each with its own set of rules and potential edge cases. This complexity increases the risk of bugs or security vulnerabilities, especially when dealing with the intricacies of each standard, like ERC721 and ERC1155's safe transfer mechanisms.

**Complexity in Compute Type Determination** - The `_determineComputeType` function determines the action (swap, deposit, withdraw) based on input and output tokens. Incorrect implementation or manipulation of this logic could lead to unintended contract behaviors, affecting liquidity operations and user funds.

**Unanticipated Interactions with Other Protocols** - Given the interconnected nature of DeFi, changes or updates in other protocols that interact with the Curve 2pool could inadvertently affect the adapter's functionality, leading to systemic risks in the broader ecosystem.

**Dependency on Accurate Token Indexing** - The contract uses `indexOf` mapping to associate Ocean IDs with their corresponding Curve pool indices. Any incorrect mapping or updates to these indices could result in failed transactions or incorrect swaps, impacting liquidity and user funds.

## Integration risks

### Protocol Interoperability

Protocol interoperability, especially in the context of the `Curve2PoolAdapter` and Ocean contracts within the DeFi ecosystem, refers to the ability of these systems to work together seamlessly. Given that DeFi protocols often rely on each other's functionalities and data, interoperability is crucial for their collective success and user trust.

### Technical Aspects of Protocol Interoperability

**Smart Contract Interface Compatibility:**

- The `Curve2PoolAdapter` and Ocean contracts must adhere to specific interfaces to interact with Curve Finance's pools and other external protocols.
- These interfaces involve function signatures, return types, and event definitions. Any mismatch in these interfaces can lead to failed transactions or incorrect data interpretation.

**Data Format and Standardization:**

- Interacting protocols must agree on data formats, such as the representation of token amounts, addresses, and transaction data.
- For instance, token amounts might need normalization to a standard decimal representation to ensure accurate calculations during swaps or liquidity provisioning.

**Handling Protocol-Specific Logic:**

- Curve Finance pools, like the 2pool, have unique mechanics, such as variable fees, slippage calculations, and liquidity dynamics.
- The `Curve2PoolAdapter` must accurately incorporate these mechanics into its logic to ensure that operations like token swaps or liquidity management reflect the underlying Curve pool's state.
- State Synchronization and Updates:
    - The state of one protocol (like liquidity levels in Curve pools) can affect the decisions and operations in another (like optimal routing in the Ocean contract).
    - Ensuring real-time or near-real-time synchronization of state information across protocols is vital for maintaining operational accuracy.

## Test Suite

The report highlights a significant discrepancy in test coverage, with certain areas like mock and ocean directories being well-tested, while others like adapters, proteus, and test/fork directories have inadequate testing.

The lack of testing in key areas such as the adapters and parts of the Proteus directory is a major concern. These untested sections could harbor undetected bugs or vulnerabilities, posing a risk to the protocol's overall security and functionality.

The high coverage in certain areas demonstrates a capability for thorough testing, which needs to be extended to the under-tested sections of the codebase.

File   |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
--------|----------|----------|----------|----------|----------|
 adapters\                            |        0 |        0 |        0 |        0 |              - |
  Curve2PoolAdapter.sol               |        0 |        0 |        0 |        0 |... 213,215,217 |
  CurveTricryptoAdapter.sol           |        0 |        0 |        0 |        0 |... 281,285,287 |
  ICurve2Pool.sol                     |      100 |      100 |      100 |      100 |              - |
  ICurveTricrypto.sol                 |      100 |      100 |      100 |      100 |              - |
  OceanAdapter.sol                    |        0 |        0 |        0 |        0 |... 153,156,157 |
 forwarder\                           |      100 |      100 |      100 |      100 |              - |
  VestingFractionalizerForwarder.sol  |      100 |      100 |      100 |      100 |              - |
 mock\                                |    92.16 |       70 |       90 |     95.6 |              - |
  ConstantSum.sol                     |      100 |      100 |      100 |      100 |              - |
  ERC1155MintsToDeployer.sol          |      100 |      100 |      100 |      100 |              - |
  ERC20MintsToDeployer.sol            |      100 |      100 |      100 |      100 |              - |
  ERC721MintsToDeployer.sol           |      100 |      100 |      100 |      100 |              - |
  Forwarder.sol                       |      100 |      100 |      100 |      100 |              - |
  Interfaces.sol                      |      100 |      100 |      100 |      100 |              - |
  MaliciousPrimitive.sol              |       50 |       50 |    71.43 |       75 |          59,85 |
  MockPrimitive.sol                   |    91.67 |       50 |     87.5 |    95.83 |            128 |
  Receive1155.sol                     |      100 |      100 |      100 |      100 |              - |
  RecursiveMaliciousPrimitive.sol     |    92.86 |       70 |     87.5 |    96.55 |            139 |
 ocean\                               |      100 |    94.77 |      100 |      100 |              - |
  BalanceDelta.sol                    |      100 |    83.33 |      100 |      100 |              - |
  ERC1155PermitSignatureExtension.sol |      100 |      100 |      100 |      100 |              - |
  IOceanFeeChange.sol                 |      100 |      100 |      100 |      100 |              - |
  IOceanPrimitive.sol                 |      100 |      100 |      100 |      100 |              - |
  IOceanToken.sol                     |      100 |      100 |      100 |      100 |              - |
  Interactions.sol                    |      100 |      100 |      100 |      100 |              - |
  Ocean.sol                           |      100 |      100 |      100 |      100 |              - |
  OceanERC1155.sol                    |      100 |     91.3 |      100 |      100 |              - |
 proteus\                             |    58.41 |       45 |    64.38 |    62.83 |              - |
  EvolvingProteus.sol                 |        0 |        0 |        0 |        0 |... 818,820,823 |
  ILiquidityPoolImplementation.sol    |      100 |      100 |      100 |      100 |              - |
  LiquidityPool.sol                   |      100 |    93.33 |      100 |      100 |              - |
  LiquidityPoolProxy.sol              |      100 |    69.44 |      100 |      100 |              - |
  Proteus.sol                         |    95.58 |    57.84 |    95.45 |    85.55 |... 691,695,697 |
  Slices.sol                          |      100 |       50 |      100 |      100 |              - |
 script\                              |        0 |      100 |        0 |        0 |              - |
  DeployCurve2PoolAdapter.s.sol       |        0 |      100 |        0 |        0 |       13,15,16 |
  DeployCurveTricryptoAdapter.s.sol   |        0 |      100 |        0 |        0 |       13,15,16 |
  DeployOcean.s.sol                   |        0 |      100 |        0 |        0 |       13,15,16 |
 test\fork\                           |        0 |        0 |        0 |        0 |              - |
  TestCurve2PoolAdapter.t.sol         |        0 |        0 |        0 |        0 |... 225,226,227 |
  TestCurveTricryptoAdapter.t.sol     |        0 |        0 |        0 |        0 |... 429,430,431 |
**All files**                             |    54.76 |     51.9 |    68.02 |    56.39 |              - |

### Recommendations

- Prioritize increasing test coverage in the adapters and proteus directories.
- Review and enhance the tests in the test/fork directory to ensure they are effectively assessing the code.
- Maintain the high level of testing in areas where coverage is already strong, ensuring ongoing robustness and reliability.

## Architecture assessment of business logic

The Ocean contract, as provided, is a comprehensive Solidity contract designed for DeFi applications, primarily focusing on the management and interaction of various token standards

![Diagram](https://gist.github.com/assets/58845085/2d6f8b65-945a-4360-8020-4e99864ace34.png)

### Core Functionalities

**Token Handling** - The contract supports interactions with multiple token types including ERC-20, ERC-721, and ERC-1155. It provides functionalities for wrapping and unwrapping these tokens. Wrapping converts tokens into a standard format within the contract, while unwrapping allows users to extract their original tokens.

**DeFi Interaction Execution** - Functions like `doInteraction` and `doMultipleInteractions` allow the contract to process both individual and batch interactions. These interactions involve executing calls to external contracts and updating the contract's state (like minting and burning tokens) based on these interactions.

**Fee Management for Unwrapping** - The contract includes a mechanism to handle fees, particularly for unwrapping tokens. This is managed through the `unwrapFeeDivisor`, which calculates the fee for unwrapping operations.

**Design and Structure** - The contract inherits from OceanERC1155 and implements interfaces such as `IOceanInteractions`, `IOceanFeeChange`, `IERC721Receiver`, and `IERC1155Receiver`. This indicates a modular design approach and compliance with ERC standards. It uses OpenZeppelin contracts for standard functionalities related to ERC token interfaces, indicating a reliance on established and tested implementations.

**Security Mechanisms** - The contract appears to incorporate reentrancy guards, as indicated by the use of constants like `NOT_INTERACTION` and `INTERACTION` and related state variables. It includes access control mechanisms, though the specific implementation details are not fully provided in the snippet.

**Events and Modifiers** - Several events are declared, which are crucial for logging and tracking transactions within the contract. Custom modifiers, like `onlyApprovedForwarder`, are used for controlling access to certain functions.

## Software engineering considerations

**Modularity and Reusability** - The contract exhibits modularity by inheriting from several interfaces and contracts, suggesting components are designed for reuse and extension. This approach aligns with solid software engineering principles, promoting maintainability and scalability.

**Design Patterns and Best Practices** - Utilization of established design patterns, such as interface-based programming and reentrancy guards, indicates adherence to industry best practices. These patterns enhance the contract’s robustness and security.

**Code Readability and Maintainability** - The contract's structure, naming conventions, and comments suggest an emphasis on readability and maintainability, crucial for long-term management and updates.

**Security** - Security is a critical aspect, especially given the DeFi context. The contract shows awareness of common vulnerabilities (like reentrancy attacks) and includes mechanisms to mitigate them. Continuous security audits and reviews are essential due to the evolving nature of smart contract exploits.

**Efficiency and Optimization** - Functions like `doMultipleInteractions` suggest a focus on optimizing transactions, likely to reduce gas costs. However, given the contract's complexity, there's a need for thorough analysis and testing to ensure efficiency, especially in gas usage.

**Error Handling and Input Validation** - The use of custom error messages and checks indicates a proactive approach to error handling and input validation, critical for robustness and user trust.

**Interoperability and Standards Compliance** - Compliance with ERC standards and the ability to interact with multiple token types demonstrate a high level of interoperability, a significant factor in the Ethereum ecosystem.

**Testing and Documentation** - Given the complexity, comprehensive testing (unit tests, integration tests) and detailed documentation are vital for ensuring the contract works as intended and to facilitate future development.

**Upgradeability** - The potential for future upgrades and modifications should be considered, especially in a rapidly evolving domain like DeFi. However, the contract doesn’t explicitly mention upgrade mechanisms.

**Dependency Management** - Reliance on external libraries like OpenZeppelin means the contract's security and functionality are partly dependent on these external components. Keeping these dependencies updated and secure is crucial.

### Time spent
20 hours

***

# Disclosures

C4 is an open organization governed by participants in the community.

C4 audits incentivize the discovery of exploits, vulnerabilities, and bugs in smart contracts. Security researchers are rewarded at an increasing rate for finding higher-risk issues. Audit submissions are judged by a knowledgeable security researcher and solidity developer and disclosed to sponsoring developers. C4 does not conduct formal verification regarding the provided code but instead provides final verification.

C4 does not provide any guarantee or warranty regarding the security of this project. All smart contract software should be used at the sole risk and responsibility of users.
