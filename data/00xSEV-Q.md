## NC-01
#### Read-only reentrancy in `_erc721Unwrap`
When `_erc721Unwrap` is called in `_doMultipleInteractions` loop it can call another contract that will read the Ocean state. The state is not fully updated. Which may lead to errors in 3rd parties similar to [Curve read-only reentrancy](https://chainsecurity.com/curve-lp-oracle-manipulation-post-mortem/)

## L-01
### Not all decimals are supported
`uint8` that holds ERC20.decimals can be up to 255. 
[Several tokens incorrectly return a uint256. If this is the case, ensure the value returned is below 255.](https://ethereum.org/ca/developers/tutorials/token-integration-checklist/#erc-conformity)

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L1138
```solidity
uint256 shift = 10 ** (uint256(decimalsTo - decimalsFrom));
convertedAmount = amountToConvert * shift;
```
There is an overflow chance for big decimals or big amounts

## L-02
### If USDC or USDT introduces a fee balances in Ocean's ledger will not equal to real balances
USDC and USDT may introduce a fee. It will lead to possible drain of the Ocean because it does not check received amount, only requested.
If amount received is less than requested the balance in Ocean's storage will be different from a real balance. After a while Ocean will become insolvent.

Consider adding support for fee on transfer tokens.

## L-03
### Not all Curve pool are supported
You can’t expect all the lending pools (and the corresponding **[DepositZaps](https://curve.readthedocs.io/exchange-deposits.html)** contracts) to implement the same API. For example, there are old and new **DepositZaps** whose methods may differ in return values or argument types.
Also, the [old](https://curve.readthedocs.io/exchange-deposits.html#deposit-zap-api-old) lending pools (and their **DepositZaps**) do not implement some of the methods that the newer pools have (e.g. **[remove_liquidity_one_coin](https://medium.com/mstable/solving-the-issue-with-slippage-in-eip-4626-3af9a5d8e597)**).

Consider checking the API when you add a new pool


