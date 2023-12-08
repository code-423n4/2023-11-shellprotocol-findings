# Low and Non-Critical

### [Low Risk](#low-risk-1)

| Total Low Risk Issues | 1 |
|:--:|:--:|

| Count | Title | Instances |
|:--:|:-------| :--: |
| [L-01](#l-01-not-updating-indexof-state-variable-for-all-coins) | Not updating `indexOf` state variable for all coins | 2 |

### [Non-Critical](#non-critical-1)

| Total Non-Critical Issues | 4 |
|:--:|:--:|

| Count | Title | Instances |
|:--:|:-------| :--: |
| [NC-01](#nc-01-redundant-comments) | Redundant comments | 2 |
| [NC-02](#nc-02-typo) | Typo | 1 |
| [NC-03](#nc-03-create-a-function-for-repeating-code) | Create a function for repeating code | 2 |
| [NC-04](#nc-04-break-up-long-and-complex-function-into-smaller-ones) | Break up long and complex function into smaller ones | 2 |

## Low Risk

### [L-01] Not updating `indexOf` state variable for all coins

`indexOf` is used to map token Ocean ids to corresponding Curve pool indices. So `coins(0)` corresponds to `indexOf[xToken] = 0` for that token. However `indexOf` is not updated for all coins.

In `Curve2PoolAdapter.sol`, for the second Curve coin `coins(1)` `indexOf` updates correctly ([here](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L86)), but for the first Curve coin `coins(0)` does not update `indexOf` correctly ([here](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L78-L82)).

Same issue occurs in `CurveTricryptoAdapter.sol`. `indexOf` for the [second](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L94) and [third](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L101) is updated, but not for the [first](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L86-L90) coin.

## Non-critical

### [NC-01] Redundant comments

There are a bunch of comments that are just stating the obvious and are just noise rather then helpful, particularly in `_doInteraction` of the `Ocean` contract.

[Code](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L398-L399)

```solidity
// Begin by unpacking the interaction type and the external contract
(InteractionType interactionType, address externalContract) = _unpackInteractionTypeAndAddress(interaction);
```

Or [code](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L407-L414)

```solidity
// Here we call _executeInteraction(), which is just a big
// if... else if... block branching on interaction type.
// Each branch sets the inputToken and outputToken and their
// respective amounts. This abstraction is what lets us treat
// interactions uniformly.
(inputToken, inputAmount, outputToken, outputAmount) = _executeInteraction(
    interaction, interactionType, externalContract, specifiedToken, interaction.specifiedAmount, userAddress
);
```

There is no point stating the obvious that we are calling the function, and explaining of what the function consists. We can simply look up the function and see that it consists of if... else if... statements.

**Recommendation:** Check the contracts and delete redundant comments to get rid of noise and unnecessary clutter. There is no need to use a comment when you can use a function or a variable. Comments should be local not explain a function in some other function.

### [NC-02] Typo

[Code](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L563)

```solidity
// burn the positive deltas from the user's balances
if (burnIds.length == 1) { ...
```

Should be:

```solidity
// burn the negative deltas from the user's balances
if (burnIds.length == 1) { ...
```

### [NC-03] Create a function for repeating code

In `Curve2PoolAdapter.sol` ([here](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L77-L95)) and in `CurveTricryptoAdapter.sol` ([here](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L85-L111)) the constructor repeats the same code 2 or 3 times. Repeating the same code multiple times is almost always bad. 

First, if we have to change something we will have to change it multiple times, instead of just once. Second, it can cause simple mistakes which are hard to find later.

**Recommendation:** Create a function for updating the storage variables for each coin. (`indexOf[token]`, `underlying[token]` and `decimals[token]`)

### [NC-04] Break up long and complex function into smaller ones

Function with multiple if... else if... statements can get very complex, long and hard to read.

In `primitiveOutputAmount()` in `Curve2PoolAdapter.sol` and `CurveTricryptoAdapter.sol` actions should be done in a separate function, for instance `_executeAction()`

Code: [here](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L162-L171) and [here](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L198-L221)