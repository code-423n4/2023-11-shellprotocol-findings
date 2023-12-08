
##  Gas Summary

| Number | Issue | Instances|
|--------|-------|----------|
|[G-01]| State variables which are not modified within functions should be set as constant or immutable for values set at deployment. or Remove if there is no used iside the contract   | 1 |
|[G-02]| Avoid Unnecessary Public Variables  | 12 |
|[G-03]| Use assembly to validate msg.sender  | 3 |
|[G-04]| Declare the variables outside the loop  | 1 |
|[G-05]| Don’t cache calls that are only used once  | 1 |
|[G-06]| Loop best practice to save gas  | 2 |
|[G-07]| Use hardcode address instead address(this)  | 11 |
|[G-08]| Counting down in for statements is more gas efficient  | 2 |
|[G-09]| Use constants instead of type(uintX).max  | 1 |


## [G-01] State variables which are not modified within functions should be set as constant or immutable for values set at deployment. or Remove if there is no used iside the contract 

```solidity
file: blob/main/src/adapters/OceanAdapter.sol

25   mapping(uint256 => address) public underlying;

```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L25

## [G-02] Avoid Unnecessary Public Variables

Public storage variables increase the contract's size due to the implicit generation of public getter functions. This makes the contract larger and could increase deployment and interaction costs.

If you do not require other contracts to read these variables, consider making them private or internal.

```solidity
file: blob/main/src/adapters/Curve2PoolAdapter.sol

56   uint256 public immutable xToken;

59   uint256 public immutable yToken;

62   uint256 public immutable lpTokenId;

```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L56


```solidity
file: blob/main/src/adapters/CurveTricryptoAdapter.sol

61   uint256 public immutable xToken;

64   uint256 public immutable yToken;

67   uint256 public immutable zToken;

70   uint256 public immutable lpTokenId;

```

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L61

```solidity
file: blob/main/src/adapters/OceanAdapter.sol

19  address public immutable ocean;

22   address public immutable primitive;

25   mapping(uint256 => address) public underlying;

```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol#L19

```solidity
file: blob/main/src/ocean/Ocean.sol

84    uint256 public immutable WRAPPED_ETHER_ID;

90    uint256 public unwrapFeeDivisor;

```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L84

## [G-03] Use assembly to validate msg.sender

We can use assembly to efficiently validate msg.sender for the didPay and uniswapV3SwapCallback functions with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs.


```solidity
file:  blob/main/src/ocean/Ocean.sol

39    require(msg.sender == ocean);

391   if (msg.value != 0) {

474   if (msg.value != 0) {

```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L39

## [G-04] Declare the variables outside the loop

Per iterations saves 26 GAS

```solidity
file: blob/main/src/ocean/Ocean.sol

501   for (uint256 i = 0; i < interactions.length;) {
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

```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L501-L540 

## [G-05] Don’t cache calls that are only used once

```solidity
file: blob/main/src/adapters/CurveTricryptoAdapter.sol

192   uint256 _balanceBefore = _getBalance(underlying[outputToken]);

```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L192

## [G-06] Loop best practice to save gas

```solidity
function Plusi() public view {
		for(uint i=0; i<10;) {
			console.log("Number ==",i);
			unchecked{
				++i;
			}
		}
	}
best practice
-----------------------------------------------------
for (uint i = 0; i < length; i = unchecked_inc(i)) {
    // do something that doesn't change the value of i
}
function unchecked_inc(uint i) returns (uint) {
    unchecked {
        return i + 1;
    }
}
```


```solidity
file: blob/main/src/ocean/Ocean.sol

463   for (uint256 i = 0; i < _idLength;) {

501   for (uint256 i = 0; i < interactions.length;) {

```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L463

## [G-07] Use hardcode address instead address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.

References:

https://book.getfoundry.sh/reference/forge-std/compute-create-address https://twitter.com/transmissions11/status/1518507047943245824

```solidity
file: blob/main/src/adapters/CurveTricryptoAdapter.sol

213   uint256 wethBalance = IERC20Metadata(underlying[zToken]).balanceOf(address(this));

216   IERC20Metadata(underlying[zToken]).balanceOf(address(this)) - wethBalance

251   return address(this).balance;

253   return IERC20Metadata(tokenAddress).balanceOf(address(this));

```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L213


```solidity
file: blob/main/src/ocean/Ocean.sol

836   SafeERC20.safeTransferFrom(IERC20(tokenAddress), userAddress, address(this), transferAmount);

891   IERC721(tokenAddress).safeTransferFrom(userAddress, address(this), tokenId);

904   IERC721(tokenAddress).safeTransferFrom(address(this), userAddress, tokenId);

929   if (tokenAddress == address(this)) revert NO_RECURSIVE_WRAPS();

931   IERC1155(tokenAddress).safeTransferFrom(userAddress, address(this), tokenId, amount, "");

964   if (tokenAddress == address(this)) revert NO_RECURSIVE_UNWRAPS();

968   IERC1155(tokenAddress).safeTransferFrom(address(this), userAddress, tokenId, amountRemaining, "");

```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L836

## [G‑08] Counting down in for statements is more gas efficient

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

by changing this logic you can save 12171 gas per one for loop 

Tools used Remix

```solidity
file: blob/main/src/ocean/Ocean.sol

463   for (uint256 i = 0; i < _idLength;) {

501   for (uint256 i = 0; i < interactions.length;) {

```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L463

### Test Code

```solidity
contract GasTest is DSTest {
    Contract0 c0;
    Contract1 c1;
    function setUp() public {
        c0 = new Contract0();
        c1 = new Contract1();
    }
    function testGas() public {
        c0.AddNum();
        c1.AddNum();
    }
}
contract Contract0 {
    uint256 num = 3;
    function AddNum() public {
        uint256 _num = num;
        for(uint i=0;i<=9;i++){
            _num = _num +1;
        }
        num = _num;
    }
}
contract Contract1 {
    uint256 num = 3;
    function AddNum() public {
        uint256 _num = num;
        for(uint i=9;i>=0;i--){
            _num = _num +1;
        }
        num = _num;
    }
}
```

## [G-09] Use constants instead of type(uintX).max

Saves 65 GAS in 5 instances.

Using constants instead of type(uintX).max saves gas in Solidity. This is because the type(uintX).max function has to dynamically calculate the maximum value of a uint256, which can be expensive in terms of gas. Constants, on the other hand, are stored in the bytecode of your contract, so they do not have to be recalculated every time you need them. Saves 13 GAS.

```solidity
file: blob/main/src/adapters/Curve2PoolAdapter.sol

190   IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);

191   IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);

```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol#L190


```solidity
file: blob/main/src/adapters/CurveTricryptoAdapter.sol

242  IERC20Metadata(tokenAddress).approve(ocean, type(uint256).max);

243  IERC20Metadata(tokenAddress).approve(primitive, type(uint256).max);

```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol#L242



```solidity
file: blob/main/src/ocean/Ocean.sol

170   unwrapFeeDivisor = type(uint256).max;

```
https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol#L170