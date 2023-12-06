## [L-01] USE OF `_MINT()`

## Impact
Ocean.sol.sol contract was using `_mint` to mint new tokens. OpenZeppelin also defines a function called `_safeMint` that prevents someone from minting ERC721 to a contract that does not support ERC721 transfers.

The `_safeMint` flavor of minting causes the recipient of the tokens, if it is a smart contract, to react upon receipt of the tokens.

https://github.com//code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L380-L430
```solidity
function _doInteraction(
    Interaction calldata interaction,
    address userAddress
)
    internal
    returns (uint256 inputToken, uint256 inputAmount, uint256 outputToken, uint256 outputAmount)
{
    // Ether payments are push only.  We always wrap ERC-X tokens using pull
    // payments, so we cannot wrap Ether using the same pattern.
    // We unwrap ERC-X tokens using push payments, so we can unwrap Ether
    // the same way.
    if (msg.value != 0) {
        inputToken = 0;
        inputAmount = 0;
        outputToken = WRAPPED_ETHER_ID;
        outputAmount = msg.value;
        emit EtherWrap(msg.value, userAddress);
    } else {
        // Begin by unpacking the interaction type and the external contract
        (InteractionType interactionType, address externalContract) = _unpackInteractionTypeAndAddress(interaction);


        // Determine the specified token based on the interaction type and the
        // interaction's external contract address, inputToken, outputToken,
        // and metadata fields. The specified token is the token
        // whose amount the user specifies.
        uint256 specifiedToken = _getSpecifiedToken(interactionType, externalContract, interaction);


        // Here we call _executeInteraction(), which is just a big
        // if... else if... block branching on interaction type.
        // Each branch sets the inputToken and outputToken and their
        // respective amounts. This abstraction is what lets us treat
        // interactions uniformly.
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
}
```

## Remediation
 use `_safeMint()` instead of `_mint()` depending on the contract’s requirements.


## [L-02] Multiple typecasts on token variables introduces unnecessary complexity and risk.


## Impact
Multiple typecasts on token variables introduce unnecessary complexity and risk. The multiple casts make the code harder to reason about and audit. Implicit assumptions could be broken.

For example, casting the token index in Curve2PoolAdapter from `int128` to `int256` then `uint256` may lead to overflows if index is negative or a loss of precision.

Similarly, packing and truncating the address down to 160-bit uint160 in OceanAdapter obscures the source and requires reasoning through wrapping effects.

And encoding the interactionType and address together in a uint256 then decoding in Ocean feels error prone.


`Curve2PoolAdapter.sol` [L167](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L167-L167)
```solidity
inputAmounts[uint256(int256(indexOfInputAmount))] = rawInputAmount;
```


`Ocean.sol` [L689](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L689-L689)
```solidity
externalContract = address(uint160(uint256(interactionTypeAndAddress)));
```

`OceanAdapter.sol` [L100](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L100-L100)
```solidity
uint256 packedValue = uint256(uint160(token));
```

## Remediation
Follow strict casting guidelines between domains with clear single steps. Add explicit overflow checks when squeezing larger types. And refactor unnecessary packing/truncation.


## [L-03] Lack of events for key token flow methods like `computeInputAmount()` reduces transparency and auditability of the Ocean and adapter contracts.

## Impact
Lack of events for key token flow methods like `computeInputAmount()` reduces transparency and auditability of the Ocean and adapter contracts.

**Impact**

* Inability to easily track or monitor `computeInputAmount` interactions off-chain
* Loss of transaction context details for debugging issues
* Reduced overall traceability into contract activity

For example, if `computeInputAmount()` in the Curve adapter was behaving incorrectly, these transactions would likely go undetected without targeted manual inspection. Preventing quick diagnosis.

`Curve2PoolAdapter.sol` [wrapToken](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L102-L114)
```solidity
    function wrapToken(uint256 tokenId, uint256 amount) internal override {
        address tokenAddress = underlying[tokenId];


        Interaction memory interaction = Interaction({
            interactionTypeAndAddress: _fetchInteractionId(tokenAddress, uint256(InteractionType.WrapErc20)),
            inputToken: 0,
            outputToken: 0,
            specifiedAmount: amount,
            metadata: bytes32(0)
        });


        IOceanInteractions(ocean).doInteraction(interaction);
    }
```
[unwrapToken](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L121-L133)
```solidity
    function unwrapToken(uint256 tokenId, uint256 amount) internal override {
        address tokenAddress = underlying[tokenId];


        Interaction memory interaction = Interaction({
            interactionTypeAndAddress: _fetchInteractionId(tokenAddress, uint256(InteractionType.UnwrapErc20)),
            inputToken: 0,
            outputToken: 0,
            specifiedAmount: amount,
            metadata: bytes32(0)
        });


        IOceanInteractions(ocean).doInteraction(interaction);
    }
```
[_approveToken](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L189-L192)
```solidity
    function _approveToken(address tokenAddress) private {
        IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);
        IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);
    }
```
`Ocean.sol` [_increaseBalanceOfPrimitive](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L1004-L1018)
```solidity
    function _increaseBalanceOfPrimitive(address primitive, uint256 inputToken, uint256 inputAmount) internal {
        // If the input token is not one of the primitive's registered tokens,
        // the primitive receives the input amount it was passed.
        // Otherwise, the tokens will be implicitly burned by the primitive
        // later in the transaction
        if (_isNotTokenOfPrimitive(inputToken, primitive) && (inputAmount > 0)) {
            // Since the primitive consented to receiving this token by not
            // reverting when it was called, we mint the token without
            // doing a safe transfer acceptance check. This breaks the
            // ERC1155 specification but in a way we hope is inconsequential, since
            // all primitives are developed by people who must be
            // aware of how the ocean works.
            _mintWithoutSafeTransferAcceptanceCheck(primitive, inputToken, inputAmount);
        }
    }
```
[_decreaseBalanceOfPrimitive](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L1038-L1046)
```solidity
    function _decreaseBalanceOfPrimitive(address primitive, uint256 outputToken, uint256 outputAmount) internal {
        // If the output token is not one of the primitive's tokens, the
        // primitive loses the output amount it just computed.
        // Otherwise, the tokens will be implicitly minted by the primitive
        // later in the transaction
        if (_isNotTokenOfPrimitive(outputToken, primitive) && (outputAmount > 0)) {
            _burn(primitive, outputToken, outputAmount);
        }
    }
```
[_grantFeeToOcean](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L1166-L1171)
```solidity
    function _grantFeeToOcean(uint256 oceanId, uint256 amount) private {
        if (amount > 0) {
            // since uint, same as (amount != 0)
            _mintWithoutSafeTransferAcceptanceCheck(owner(), oceanId, amount);
        }
    }
```

`CurveTricryptoAdapter.sol` [wrapToken](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L118-L140)
```solidity
    function wrapToken(uint256 tokenId, uint256 amount) internal override {
        Interaction memory interaction;


        if (tokenId == zToken) {
            interaction = Interaction({
                interactionTypeAndAddress: 0,
                inputToken: 0,
                outputToken: 0,
                specifiedAmount: 0,
                metadata: bytes32(0)
            });
            IOceanInteractions(ocean).doInteraction{ value: amount }(interaction);
        } else {
            interaction = Interaction({
                interactionTypeAndAddress: _fetchInteractionId(underlying[tokenId], uint256(InteractionType.WrapErc20)),
                inputToken: 0,
                outputToken: 0,
                specifiedAmount: amount,
                metadata: bytes32(0)
            });
            IOceanInteractions(ocean).doInteraction(interaction);
        }
    }
```
[unwrapToken](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L147-L169)
```solidity
    function unwrapToken(uint256 tokenId, uint256 amount) internal override {
        Interaction memory interaction;


        if (tokenId == zToken) {
            interaction = Interaction({
                interactionTypeAndAddress: _fetchInteractionId(address(0), uint256(InteractionType.UnwrapEther)),
                inputToken: 0,
                outputToken: 0,
                specifiedAmount: amount,
                metadata: bytes32(0)
            });
        } else {
            interaction = Interaction({
                interactionTypeAndAddress: _fetchInteractionId(underlying[tokenId], uint256(InteractionType.UnwrapErc20)),
                inputToken: 0,
                outputToken: 0,
                specifiedAmount: amount,
                metadata: bytes32(0)
            });
        }


        IOceanInteractions(ocean).doInteraction(interaction);
    }
```
[_approveToken](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L241-L244)
```solidity
    function _approveToken(address tokenAddress) private {
        IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);
        IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);
    }
```
[computeOutputAmount](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L55-L76)
```solidity
    function computeOutputAmount(
        uint256 inputToken,
        uint256 outputToken,
        uint256 inputAmount,
        address,
        bytes32 metadata
    )
        external
        override
        onlyOcean
        returns (uint256 outputAmount)
    {
        unwrapToken(inputToken, inputAmount);


        // handle the unwrap fee scenario
        uint256 unwrapFee = inputAmount / IOceanInteractions(ocean).unwrapFeeDivisor();
        uint256 unwrappedAmount = inputAmount - unwrapFee;


        outputAmount = primitiveOutputAmount(inputToken, outputToken, unwrappedAmount, metadata);


        wrapToken(outputToken, outputAmount);
    }
```
[computeInputAmount](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L81-L94)
```solidity
    function computeInputAmount(
        uint256 inputToken,
        uint256 outputToken,
        uint256 outputAmount,
        address userAddress,
        bytes32 maximumInputAmount
    )
        external
        override
        onlyOcean
        returns (uint256 inputAmount)
    {
        revert();
    }
```

While workarounds like transaction analysis are possible, the lack of events adds overhead and uncertainty.

## Remediation
Adding events for key token interactions would significantly improve transparency and auditing. Making it easy to trace transactions on Curve adapter integrations.

## [L-04] Function return value in the signature that doesn't match the actual returns is misleading and introduces potential logic issues.

## Impact
This function `function onERC721Received`, `function onERC1155Received` specifies a `returns` keyword in the function signature but does not mention what to return anywhere in the function. This forces the function to always return a default value that was specified in the signature despite the calculations inside the function.

The mismatch between specified and actual returns can lead to unintended behavior if the caller expects data that is not provided.

For example, if `onERC1155Received()` had a complex return value declared, but returned a simple boolean, callers may improperly rely on complex encoded data that isn't there.

[function onERC721Received](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L309-L315)
```solidity
function onERC721Received(address, address, uint256, bytes calldata) external view override returns (bytes4) {
    if (_ERC721InteractionStatus == INTERACTION) {
        return IERC721Receiver.onERC721Received.selector;
    } else {
        return 0;
    }
}
```
[function onERC1155Received](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L326-L343)
```solidity
function onERC1155Received(
    address,
    address,
    uint256,
    uint256,
    bytes calldata
)
    external
    view
    override
    returns (bytes4)
{
    if (_ERC1155InteractionStatus == INTERACTION) {
        return IERC1155Receiver.onERC1155Received.selector;
    } else {
        return 0;
    }
}
```
## Remediation
Aligning specified returns with actual return statements is best practice to avoid confusion.

## [NC-01] Missing visibility modifiers for state variables reduces clarity around intended access levels and introduces potential risks.

## Impact
Visibility modifiers determine the level of access to the variables in your smart contract. This defines the level of access for contracts and other external users. It makes it easier to understand who can access the variable.The contract defined a state variable decimals that was missing a visibility modifier. The lack of specified visibility means these variables default to `internal`. This makes their read/write access implicit based on contract relationships rather than explicit intents.

If they were meant to be `private` or `public`, this could lead to unintended exposure or break required access needs.

If `decimals` was meant to be public but lacked the modifier, off-chain clients would have their access break without realizing why.

Or if `_ERC1155InteractionStatus` was meant to be private but left internal, malicious contracts that inherit Ocean could manipulate it.

`Curve2PoolAdapter.sol` [Line 65](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L65-L65)

```solidity
mapping(uint256 => int128) indexOf;
```
[Line 68](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L68-L68)
```solidity
mapping(uint256 => uint8) decimals;
```

`Ocean.sol` [Line 108](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L108-L108)
```solidity
uint256 _ERC1155InteractionStatus;
```
[Line 109](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L109-L109)
```solidity
uint256 _ERC721InteractionStatus;
```

`CurveTricryptoAdapter.sol` [Line 73](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L73-L73)
```solidity
mapping(uint256 => uint256) indexOf;
```
[Line 76](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L76-L76)
```solidity
mapping(uint256 => uint8) decimals;
```

## Remediation
Adding explicit visibility modifiers would capture intent and prevent confusion over intended access control.

## [NC-02] Using `require()` for validation is generally preferable to using `revert()` directly for logic checks, due to the additional clarity require provides. 

## Impact
Using `revert()` statements for validation logic directly makes it harder to understand the intended conditional check vs actual revert reasons. It reduces readability.

And `revert()` doesn't allow passing revert reason strings - losing context on failures. The logic still functions for checks. But it inhibits understanding and support.

As an example, if the fee divisor check failed, tooling and analytics would only see a generic revert rather than understanding an invalid fee threshold was passed.

Ocean.sol [Line 198](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L198-L198)
```solidity
if (MIN_UNWRAP_FEE_DIVISOR > nextUnwrapFeeDivisor) revert();
```

## Remediation
Switching to clearly defined `require()` statements for validation would improve readability through both encapsulated conditional logic and custom revert reasons.

## [NC-03] Using `.transfer()` or `.send()` instead of `.call()` does introduce risks when sending ether.

## Impact
Using `transfer` or `send` function call this is unsafe as `transfer` has hard coded gas budget and can fail if the user is a smart contract `.transfer()` only forwards 2300 gas which may be insufficient for recipient contracts to handle the payment. This can cause the payment to get stuck.

And `.send()` provides no gas stipend at all, making failure even more likely.

This could prevent users from receiving expected payments if their wallet relies on custom logic.

`Ocean.sol` [Line 982](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L982-L982)
```solidity
payable(userAddress).transfer(transferAmount);
```

## Remediation
Switching to `.call(){}` handles forwarding sufficient gas for recipients while retaining revert protection.


## [NC-04] Defining an empty fallback function without clear intent introduces confusion and potential issues.

## Impact
An empty fallback could lead to confusion for users sending ether - they may expect it to be used meaningfully or trigger logic. It also increases deployment sizes with unused code.

And not reverting falls outside best practice guidance.

`CurveTricryptoAdapter.sol` [Line 291](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L291-L291)
```solidity
fallback() external payable { }
```
If users send ETH to Curve adapters expecting swap or deposit behavior, the lack of revert would lead to confusion when nothing occurs.

Or if the fallback usage expectations change later, failure to revert could lead to issues handling unconsidered payloads.

## Remediation
Reverting with custom errors explaining fallback intent would reduce ambiguity. As would inline comments detailing expectations.
