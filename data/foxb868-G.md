
## [GAS-01] Using `abi.encode()` instead of `abi.encodePacked()` can lead to unnecessary gas usage from padded data.

## Impact
The padding and length prefixes in `abi.encode()` result in larger `calldata` and memory footprint for encoding data. This bloats transactions and increases gas costs.

For contract systems doing frequent encoding, these compounding costs can be sizable.

```solidity
return bytes32(abi.encode(packedValue));
```
https://github.com//code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L102-L102

As a numerical example, encoding 4 bytes with `abi.encode()` uses 32 bytes with 28 bytes of padding versus just the 4 bytes needed with `abi.encodePacked()`.

## Recommendation
Switching to `abi.encodePacked()` would tightly pack the data removing the padding and length bloat.

## [GAS-02] Defining constructors as payable can lead to modest gas savings as you pointed out. Let's discuss the tradeoffs.

## Impact
Making constructors payable saves about 10 opcodes and some gas by removingchecks. This leads to savings on contract deployment.

However, it allows sending ether during deployment which is often undesirable.

There are no gas savings after deployment. Only deploy cost.

For a contract deployed 100 times, each deployment might save 5,000 gas * 100 = 500,000 total gas saved.

```solidity
constructor(address ocean_, address primitive_) OceanAdapter(ocean_, primitive_) {
    address xTokenAddress = ICurve2Pool(primitive).coins(0);
    xToken = _calculateOceanId(xTokenAddress, 0);
    underlying[xToken] = xTokenAddress;
    decimals[xToken] = IERC20Metadata(xTokenAddress).decimals();
    _approveToken(xTokenAddress);


    address yTokenAddress = ICurve2Pool(primitive).coins(1);
    yToken = _calculateOceanId(yTokenAddress, 0);
    indexOf[yToken] = int128(1);
    underlying[yToken] = yTokenAddress;
    decimals[yToken] = IERC20Metadata(yTokenAddress).decimals();
    _approveToken(yTokenAddress);


    lpTokenId = _calculateOceanId(primitive_, 0);
    underlying[lpTokenId] = primitive_;
    decimals[lpTokenId] = IERC20Metadata(primitive_).decimals();
    _approveToken(primitive_);
}
```
[Curve2PoolAdapter.sol#L77-L95](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L77-L95)

```solidity
constructor(string memory uri_) OceanERC1155(uri_) {
    unwrapFeeDivisor = type(uint256).max;
    _ERC1155InteractionStatus = NOT_INTERACTION;
    _ERC721InteractionStatus = NOT_INTERACTION;
    WRAPPED_ETHER_ID = _calculateOceanId(address(0x4574686572), 0); // hexadecimal(ascii("Ether"))
}
```
[Ocean.sol#L169-L174](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L169-L174)

```solidity
constructor(address ocean_, address primitive_) OceanAdapter(ocean_, primitive_) {
    address xTokenAddress = ICurveTricrypto(primitive).coins(0);
    xToken = _calculateOceanId(xTokenAddress, 0);
    underlying[xToken] = xTokenAddress;
    decimals[xToken] = IERC20Metadata(xTokenAddress).decimals();
    _approveToken(xTokenAddress);


    address yTokenAddress = ICurveTricrypto(primitive).coins(1);
    yToken = _calculateOceanId(yTokenAddress, 0);
    indexOf[yToken] = 1;
    underlying[yToken] = yTokenAddress;
    decimals[yToken] = IERC20Metadata(yTokenAddress).decimals();
    _approveToken(yTokenAddress);


    address wethAddress = ICurveTricrypto(primitive).coins(2);
    zToken = _calculateOceanId(address(0x4574686572), 0); // hexadecimal(ascii("Ether"))
    indexOf[zToken] = 2;
    underlying[zToken] = wethAddress;
    decimals[zToken] = NORMALIZED_DECIMALS;
    _approveToken(wethAddress);


    address lpTokenAddress = ICurveTricrypto(primitive).token();
    lpTokenId = _calculateOceanId(lpTokenAddress, 0);
    underlying[lpTokenId] = lpTokenAddress;
    decimals[lpTokenId] = IERC20Metadata(lpTokenAddress).decimals();
    _approveToken(lpTokenAddress);
}
```
[CurveTricryptoAdapter.sol#L85-L111](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L85-L111)

## Recommendation
Making constructors payable makes sense for systems encouraging/handling ether sending during deployment. Otherwise reverting payable constructors is best practice.

## [GAS-03] Using generic `revert` statements instead of custom errors misses opportunities for gas savings and improved debugging.

## Impact
Basic `revert` usage emits the static 0xfe error opcode, costing around `700 gas`.

Whereas custom errors can set dynamic reason strings for `200-300 gas`, saving `400+ gas` per check.

This can lead to substantial gas optimizations for contracts doing frequent validation.

```solidity
if (MIN_UNWRAP_FEE_DIVISOR > nextUnwrapFeeDivisor) revert();
```
[Ocean.sol#L198](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L198-L198)

```solidity
        revert();
```
[OceanAdapter.sol#L93](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L93-L93)

If the `MIN_UNWRAP_FEE_DIVISOR` check reverts 100 times, that's 100 * (700 - 300) = 40,000 gas wasted by using a generic `revert` over a custom string error.

## Recommendation
Defining clear custom error types and using them for validation improves understanding and saves gas.

## [GAS-04] Using strict inequality checks can lead to unnecessary gas costs compared to non-strict versions.

## Impact
Solidity uses `JUMPI` for strict inequality jumps costing 10 gas, whereas non-strict uses cheaper `JUMP` costing only 8 gas.

So each strict inequality check wastes 2 gas that can add up substantially.

```solidity
if (uint256(minimumOutputAmount) > outputAmount) revert SLIPPAGE_LIMIT_EXCEEDED();
```
[Curve2PoolAdapter.sol#L175](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L175-L175)

```solidity
if (MIN_UNWRAP_FEE_DIVISOR > nextUnwrapFeeDivisor) revert();
```
[Ocean.sol#L198](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L198-L198)

```solidity
if (inputAmount > 0) {
```
[Ocean.sol#L419](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L419-L419)

```solidity
if (outputAmount > 0) {
```
[Ocean.sol#L426](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L426-L426)

```solidity
if (_isNotTokenOfPrimitive(inputToken, primitive) && (inputAmount > 0)) {
```
[Ocean.sol#L1009](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L1009-L1009)

```solidity
if (_isNotTokenOfPrimitive(outputToken, primitive) && (outputAmount > 0)) {
```
[Ocean.sol#L1043](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L1043-L1043)

```solidity
if (truncated > 0) {
```
[Ocean.sol#L1084](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L1084-L1084)

```solidity
if (amount > 0) {
```
[Ocean.sol#L1167](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L1167-L1167)

```solidity
if (uint256(minimumOutputAmount) > outputAmount) revert SLIPPAGE_LIMIT_EXCEEDED();
```
[CurveTricryptoAdapter.sol#L227](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L227-L227)

With 8 inequality checks, each execution wastes `8 * 2 = 16 gas`.

If executed 100 times, `100 * 16 = 1,600 gas` is wasted.

## Recommendation
Switching to non-strict inequalities where logic allows would provide direct gas savings.


## [GAS-05] Returning multiple values from the `doInteraction` and `doMultipleInteractions` functions instead of a struct make the code harder to read and reason about.

## Impact
Having multiple return values forces callers to keep track of the order and meaning of each one instead of accessing named struct fields. This cognitive load can lead to bugs from mismatching.

It also makes quickly discerning the main outputs of these functions more difficult when reading the code.

**Maintenance Overhead**

If additional return values are needed in the future, each caller has to correctly manage those as well in the right order instead of simply accessing a new struct field. This complicates modifications.

```solidity
function doInteraction(Interaction calldata interaction)
external
payable
override
returns (uint256 burnId, uint256 burnAmount, uint256 mintId, uint256 mintAmount)
{
emit OceanTransaction(msg.sender, 1);
return _doInteraction(interaction, msg.sender);
}
```
[Ocean.sol#doInteraction](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L210-L218)


```solidity
function doMultipleInteractions(
    Interaction[] calldata interactions,
    uint256[] calldata ids
)
    external
    payable
    override
    returns (
        uint256[] memory burnIds,
        uint256[] memory burnAmounts,
        uint256[] memory mintIds,
        uint256[] memory mintAmounts
    )
{
    emit OceanTransaction(msg.sender, interactions.length);
    return _doMultipleInteractions(interactions, ids, msg.sender);
}
```
[Ocean.sol#doMultipleInteractions](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L229-L245)

```solidity
function forwardedDoInteraction(
    Interaction calldata interaction,
    address userAddress
)
    external
    payable
    override
    onlyApprovedForwarder(userAddress)
    returns (uint256 burnId, uint256 burnAmount, uint256 mintId, uint256 mintAmount)
{
    emit ForwardedOceanTransaction(msg.sender, userAddress, 1);
    return _doInteraction(interaction, userAddress);
}
```
[Ocean.sol#forwardedDoInteraction](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L256-L268)


```solidity
function forwardedDoMultipleInteractions(
    Interaction[] calldata interactions,
    uint256[] calldata ids,
    address userAddress
)
    external
    payable
    override
    onlyApprovedForwarder(userAddress)
    returns (
        uint256[] memory burnIds,
        uint256[] memory burnAmounts,
        uint256[] memory mintIds,
        uint256[] memory mintAmounts
    )
{
    emit ForwardedOceanTransaction(msg.sender, userAddress, interactions.length);
    return _doMultipleInteractions(interactions, ids, userAddress);
}
```
[Ocean.sol#forwardedDoMultipleInteractions](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L281-L299)


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
[Ocean.sol#_doInteraction](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L380-L430)


```solidity
function _doMultipleInteractions(
    Interaction[] calldata interactions,
    uint256[] calldata ids,
    address userAddress
)
    internal
    returns (
        uint256[] memory burnIds,
        uint256[] memory burnAmounts,
        uint256[] memory mintIds,
        uint256[] memory mintAmounts
    )
{
    // Use the passed ids to create an array of balance deltas, used in
    // the intra-transaction accounting system.
    BalanceDelta[] memory balanceDeltas = new BalanceDelta[](ids.length);


    uint256 _idLength = ids.length;
    for (uint256 i = 0; i < _idLength;) {
        balanceDeltas[i] = BalanceDelta(ids[i], 0);
        unchecked {
            ++i;
        }
    }


    // Ether payments are push only.  We always wrap ERC-X tokens using pull
    // payments, so we cannot wrap Ether using the same pattern.
    // We unwrap ERC-X tokens using push payments, so we can unwrap Ether
    // the same way.
    if (msg.value != 0) {
        // If msg.value != 0 and the user did not pass the WRAPPED_ETHER_ID
        // as an id in the ids array, the balance delta library will revert
        // This protects users who accidentally provide a msg.value.
        balanceDeltas.increaseBalanceDelta(WRAPPED_ETHER_ID, msg.value);
        emit EtherWrap(msg.value, userAddress);
    }


    // Execute the interactions
    {
        /**
         * @dev Solidity does not reuse memory that has gone out of scope
         *  and the gas cost of memory usage grows quadratically.
         * @dev We passed interactions as calldata to lower memory usage.
         *  However, accessing the members of a calldata structure uses
         *  more local identifiers than accessing the members of an
         *  in-memory structure. We're right up against the limit on
         *  local identifiers. To solve this, we allocate a single
         *  structure in memory and copy the calldata structures over one
         *  by one as we process them.
         */
        Interaction memory interaction;
        // This pulls the user's address to the top of the stack, above
        // the ids array, which we won't need again. We're right up against
        // the locals limit and this does the trick. Is there a better way?
        address userAddress_ = userAddress;


        for (uint256 i = 0; i < interactions.length;) {
            interaction = interactions[i];


            (InteractionType interactionType, address externalContract) =
                _unpackInteractionTypeAndAddress(interaction);


            // specifiedToken is the token whose amount the user specifies
            uint256 specifiedToken = _getSpecifiedToken(interactionType, externalContract, interaction);


            // A user can pass uint256.max as the specifiedAmount when they
            // want to use the total amount of the token held in the
            // balance delta. Otherwise, the specifiedAmount is just the
            // amount the user passed for this interaction.
            uint256 specifiedAmount;
            if (interaction.specifiedAmount == GET_BALANCE_DELTA) {
                specifiedAmount = balanceDeltas.getBalanceDelta(interactionType, specifiedToken);
            } else {
                specifiedAmount = interaction.specifiedAmount;
            }


            (uint256 inputToken, uint256 inputAmount, uint256 outputToken, uint256 outputAmount) =
            _executeInteraction(
                interaction, interactionType, externalContract, specifiedToken, specifiedAmount, userAddress_
            );


            // inputToken is given up by the user during the interaction
            if (inputAmount > 0) {
                // equivalent to (inputAmount != 0)
                balanceDeltas.decreaseBalanceDelta(inputToken, inputAmount);
            }


            // outputToken is gained by the user during the interaction
            if (outputAmount > 0) {
                // equivalent to (outputAmount != 0)
                balanceDeltas.increaseBalanceDelta(outputToken, outputAmount);
            }
            unchecked {
                ++i;
            }
        }
    }


    // Persist intra-transaction balance deltas to the Ocean's public ledger
    {
        // Place positive deltas into mintIds and mintAmounts
        // Place negative deltas into burnIds and burnAmounts
        (mintIds, mintAmounts, burnIds, burnAmounts) = balanceDeltas.createMintAndBurnArrays();


        // Here we should know that uint[] memory arr = new uint[](0);
        // produces a reference to an empty array called arr with property
        // (arr.length == 0)


        // mint the positive deltas to the user's balances
        if (mintIds.length == 1) {
            // if there's only one we can just use the more semantically
            // appropriate _mint
            _mint(userAddress, mintIds[0], mintAmounts[0]);
        } else if (mintIds.length > 1) {
            // if there's more than one we use _mintBatch
            _mintBatch(userAddress, mintIds, mintAmounts);
        } // if there are none, we do nothing


        // burn the positive deltas from the user's balances
        if (burnIds.length == 1) {
            // if there's only one we can just use the more semantically
            // appropriate _burn
            _burn(userAddress, burnIds[0], burnAmounts[0]);
        } else if (burnIds.length > 1) {

```
[Ocean.sol#_doMultipleInteractions](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L445-L573)


```solidity
function _executeInteraction(
    Interaction memory interaction,
    InteractionType interactionType,
    address externalContract,
    uint256 specifiedToken,
    uint256 specifiedAmount,
    address userAddress
)
    internal
    returns (uint256 inputToken, uint256 inputAmount, uint256 outputToken, uint256 outputAmount)
{
    if (interactionType == InteractionType.ComputeOutputAmount) {
        inputToken = specifiedToken;
        inputAmount = specifiedAmount;
        outputToken = interaction.outputToken;
        outputAmount = _computeOutputAmount(
            externalContract, inputToken, outputToken, inputAmount, userAddress, interaction.metadata
        );
    } else if (interactionType == InteractionType.ComputeInputAmount) {
        inputToken = interaction.inputToken;
        outputToken = specifiedToken;
        outputAmount = specifiedAmount;
        inputAmount = _computeInputAmount(
            externalContract, inputToken, outputToken, outputAmount, userAddress, interaction.metadata
        );
    } else if (interactionType == InteractionType.WrapErc20) {
        inputToken = 0;
        inputAmount = 0;
        outputToken = specifiedToken;
        outputAmount = specifiedAmount;
        _erc20Wrap(externalContract, outputAmount, userAddress, outputToken);
    } else if (interactionType == InteractionType.UnwrapErc20) {
        inputToken = specifiedToken;
        inputAmount = specifiedAmount;
        outputToken = 0;
        outputAmount = 0;
        _erc20Unwrap(externalContract, inputAmount, userAddress, inputToken);
    } else if (interactionType == InteractionType.WrapErc721) {
        // An ERC-20 or ERC-1155 contract can have a transfer with
        // any amount including zero. Here, we need to require that
        // the specifiedAmount is equal to one, since the external
        // call to the ERC-721 contract does not include an amount,
        // and the ledger is mutated based on the specifiedAmount.
        if (specifiedAmount != 1) revert INVALID_ERC721_AMOUNT();
        inputToken = 0;
        inputAmount = 0;
        outputToken = specifiedToken;
        outputAmount = specifiedAmount;
        _erc721Wrap(externalContract, uint256(interaction.metadata), userAddress, outputToken);
    } else if (interactionType == InteractionType.UnwrapErc721) {
        // See the comment in the preceeding else if block.
        if (specifiedAmount != 1) revert INVALID_ERC721_AMOUNT();
        inputToken = specifiedToken;
        inputAmount = specifiedAmount;
        outputToken = 0;
        outputAmount = 0;
        _erc721Unwrap(externalContract, uint256(interaction.metadata), userAddress, inputToken);
    } else if (interactionType == InteractionType.WrapErc1155) {
        inputToken = 0;
        inputAmount = 0;
        outputToken = specifiedToken;
        outputAmount = specifiedAmount;
        _erc1155Wrap(externalContract, uint256(interaction.metadata), outputAmount, userAddress, outputToken);
    } else if (interactionType == InteractionType.UnwrapErc1155) {
        inputToken = specifiedToken;
        inputAmount = specifiedAmount;
        outputToken = 0;
        outputAmount = 0;
        _erc1155Unwrap(externalContract, uint256(interaction.metadata), inputAmount, userAddress, inputToken);
    } else {
        assert(interactionType == InteractionType.UnwrapEther && specifiedToken == WRAPPED_ETHER_ID);
        inputToken = specifiedToken;
        inputAmount = specifiedAmount;
        outputToken = 0;
        outputAmount = 0;
        _etherUnwrap(inputAmount, userAddress);
    }
}
```
[Ocean.sol#_executeInteraction](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L597-L674)

## Recommendation
Defining a struct encapsulating the key output values with named fields would significantly improve readability and maintainability.

## [GAS-06] Declaring variables like NORMALIZED_DECIMALS that are never read introduces confusion and potential gas costs.

## Impact
Unused variables bloat contract size, costing additional deployment gas that is wasted. And they signal intent to use that value, so when it is not actually referenced, this leads to confusion.

```solidity
uint8 constant NORMALIZED_DECIMALS = 18;
```
[OceanAdapter.sol#L16](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L16-L16)

While not overtly exploitable, dead code generally signals gaps in quality assurance and review processes.

## Recommendation
Removing unused variable declarations where possible improves correctness, understandability and efficiency.


## [GAS-07] Using `>` conditional checks on unsigned integers instead of `!= 0` can lead to unnecessary gas costs.

## Impact
For unsigned types, Solidity compiles `x > 0` to a GT opcode which costs 3 gas, whereas `x != 0` compiles to an `ISZERO` followed by `ISZERO` costing only 2 gas.

So each `> 0` check wastes 1 gas that adds up.

```solidity
if (inputAmount > 0) {
```
[Ocean.sol#L419](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L419-L419)


```solidity
if (outputAmount > 0) {
```
[Ocean.sol#L426](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L426-L426)


```solidity
if (_isNotTokenOfPrimitive(inputToken, primitive) && (inputAmount > 0)) {
```
[Ocean.sol#L1009](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L1009-L1009)


```solidity
if (_isNotTokenOfPrimitive(outputToken, primitive) && (outputAmount > 0)) {
```
[Ocean.sol#L1043](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L1043-L1043)


```solidity
if (truncated > 0) {
```
[Ocean.sol#L1084](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L1084-L1084)


```solidity
if (amount > 0) {
```
[Ocean.sol#L1167](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L1167-L1167)

**Example**

With 6 checks, each execution wastes `6 * 1 = 6 gas`.

If executed 100 times, `100 * 6 = 600` gas is wasted.

## Recommendation
Switching to `!= 0` where logic allows would provide direct gas savings.


## [GAS-08] Frequently accessing state variables triggers multiple `SLOADs` which can be gas inefficient compared to caching values in memory using `MLOAD/MSTORE`.

## Impact
State variable `SLOADs` cost 200 gas after the first one, whereas memory access is only 3 gas. So frequent `SLOADs` waste gas. For code hot paths, this adds up quickly.

```solidity
uint256 public immutable xToken;
```
[Curve2PoolAdapter.sol#L56](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L56-L56)

```solidity
uint256 public immutable yToken;
```
[Curve2PoolAdapter.sol#L59](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L59-L59)

```solidity
uint256 public immutable lpTokenId;
```
[Curve2PoolAdapter.sol#L62](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L62-L62)

```solidity
mapping(uint256 => uint8) decimals;
```
[Curve2PoolAdapter.sol#L68](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L68-L68)

```solidity
uint256 public immutable xToken;
```
[Curve2PoolAdapter.sol#L56](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L56-L56)

```solidity
uint256 public immutable yToken;
```
[Curve2PoolAdapter.sol#L59](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L59-L59)

```solidity
uint256 public immutable lpTokenId;
```
[Curve2PoolAdapter.sol#L62](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L62-L62)

```solidity
uint256 constant NOT_INTERACTION = 1;
```
[Ocean.sol#L106](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L106-L106)

```solidity
uint256 public unwrapFeeDivisor;
```
[Ocean.sol#L90](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L90-L90)

```solidity
uint256 _ERC721InteractionStatus;
```
[Ocean.sol#L109](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L109-L109)

Example

If `xToken` is read 100 times in a function called 100 times, that’s 100 * 99 * (200-3) = ~20,000 wasted gas from redundant `SLOADs`.

## Recommendation
Caching frequently read state variables in memory at function start minimizes redundant `SLOADs`.