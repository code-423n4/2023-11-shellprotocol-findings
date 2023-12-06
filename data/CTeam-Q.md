1.

```solidity
   function _etherUnwrap(uint256 amount, address userAddress) private {
        uint256 feeCharged = _calculateUnwrapFee(amount);
        _grantFeeToOcean(WRAPPED_ETHER_ID, feeCharged);
        uint256 transferAmount = amount - feeCharged;
        payable(userAddress).send(transferAmount);

        emit EtherUnwrap(transferAmount, feeCharged, userAddress);
    }
```

It is possible for a function to charge a fee from a small amount, where the end result is that the total transfer value equals 0, but the fee is greater than 0, without causing the function to revert.

To fix this, ensure amount > 0 && amount >= fee 

2.

```soidity
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

The unwrapFee in the smart contract can theoretically be zero if unwrapFeeDivisor() larger than the inputAmount. 

To fix this, ensure unwrappedAmount > 0

3.

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
        // ... Other code
```

Here, there is no limit to the number of interactions array value.

In Solidity, when a loop in a smart contract reaches the maximum gas limit allocated for a transaction, the Ethereum Virtual Machine (EVM) will halt the execution of that transaction. 

To fix this, we should restrict the number of interaction number to a magic number.

4.

There are some typos in Ocean.sol

Suporting should be supporting
Exection should be execution
Recieve should be receive
Imlementation should be implementation
Modifer should be modifier
Prefering should be preferring
Ouput should be output
Preceeding should be preceding
Transferrable should be transferable

5.

_doMultipleInteractions() loops through a series of ERC20, ERC721 or ERC1155 token interactions to calculate final balances. 

However, there is a robustness issue currently in that if any single external call fails or has a bug, the entire _doMultipleInteractions transaction will fail and roll back. This is because there is no error handling around those external calls.

To fix this, suggest to wrap each interaction's external token transfer call with try-catch. The benefit is that a failure in one external token contract won't necessarily doom the processing of all the other interactions. We compartmentalize the risk to each individual external call. This follows a good practice of error isolation and atomicity in code design.

6.

```solidity
  function _computeOutputAmount(
        address primitive,
        uint256 inputToken,
        uint256 outputToken,
        uint256 inputAmount,
        address userAddress,
        bytes32 metadata
    )
        internal
        returns (uint256 outputAmount)
    {
        // mint before making a external call to the primitive to integrate with external protocol primitive adapters
        _increaseBalanceOfPrimitive(primitive, inputToken, inputAmount);

        outputAmount =
            IOceanPrimitive(primitive).computeOutputAmount(inputToken, outputToken, inputAmount, userAddress, metadata);

        _decreaseBalanceOfPrimitive(primitive, outputToken, outputAmount);

        emit ComputeOutputAmount(primitive, inputToken, outputToken, inputAmount, outputAmount, userAddress);
    }
```

The variable address primitive here is misleading, it refers to adapter. Primitive could not have computeOutputAmount function. 

See Ocean#computeOutputAmount

To fix this, refactor primitive to adapter

