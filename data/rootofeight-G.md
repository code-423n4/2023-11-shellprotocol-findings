#### Default Variable Values and Gas Efficiency in Smart Contracts

If a variable is not set/initialized, it is assumed to have the default value(0, false, 0x0 etc depending on the data type). Explicitly initializing it with its default value is an anti-pattern and wastes gas.

Source : https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L501C18-L501C31

```
for (uint256 i = 0; i < interactions.length;)

change to for (uint256 i; i < interactions.length;)

```

#### Calculate length and compare to save gas

When working with arrays or strings in smart contracts, it's more gas-efficient to calculate their length once and store it in a variable. Instead of repeatedly calculating the length in loops or comparisons, which can consume extra gas, storing the length beforehand helps to reduce computational costs and optimize gas usage in contract operations.

Source: https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L501C18-L501C31


