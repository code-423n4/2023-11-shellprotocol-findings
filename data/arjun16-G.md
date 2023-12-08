# G-01. abi.encode() is less efficient than abi.encodepacked() #

In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.

Reference : https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison
```
file : src/adapters
/OceanAdapter.sol

102 :         return bytes32(abi.encode(packedValue));
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/OceanAdapter.sol#L102

# G-02. Event : Use “indexed” keyword for uint, bool, and address #

Using the indexed keyword for value types such as uint, bool, and address saves gas costs, as seen in the example below. However, this is only the case for value types, whereas indexing bytes and strings are more expensive than their unindexed version.
Also indexed keyword has more merits.

It can be useful to have a way to monitor the contract’s activity after it was deployed. One way to accomplish this is to look at all transactions of the contract, however that may be insufficient, as message calls between contracts are not recorded in the blockchain. Moreover, it shows only the input parameters, not the actual changes being made to the state. Also events could be used to trigger functions in the user interface.

Reference : https://0xmacro.com/blog/solidity-gas-optimizations-cheat-sheet/
```
file : src/ocean
/Ocean.sol

142 :     event EtherUnwrap(uint256 amount, uint256 feeCharged, address indexed user);

145 :         uint256 inputToken,

153 :         uint256 inputToken,

159 :     event OceanTransaction(address indexed user, uint256 numberOfInteractions);

160 :     event ForwardedOceanTransaction(address indexed forwarder, address indexed user, uint256 numberOfInteractions);

```
https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L142

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L145

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L153

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L159

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L160
```
file : src/adapters
/CurveTricryptoAdapter.sol

35 - 41 :     event Swap(
        uint256 inputToken,
        uint256 inputAmount,
        uint256 outputAmount,
        bytes32 slippageProtection,
        address user,
        bool computeOutput

43 - 49 :     event Deposit(
        uint256 inputToken,
        uint256 inputAmount,
        uint256 outputAmount,
        bytes32 slippageProtection,
        address user,
        bool computeOutput

51 - 57 :     event Withdraw(
        uint256 outputToken,
        uint256 inputAmount,
        uint256 outputAmount,
        bytes32 slippageProtection,
        address user,
        bool computeOutput
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L35C5-L41C13

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L43C5-L49C12

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L51C5-L57C11
```
file : src/adapters
/Curve2PoolAdapter.sol

30 - 36 :     event Swap(
        uint256 inputToken,
        uint256 inputAmount,
        uint256 outputAmount,
        bytes32 slippageProtection,
        address user,
        bool computeOutput

38 - 44 :     event Deposit(
        uint256 inputToken,
        uint256 inputAmount,
        uint256 outputAmount,
        bytes32 slippageProtection,
        address user,
        bool computeOutput

46 - 52 :     event Withdraw(
        uint256 outputToken,
        uint256 inputAmount,
        uint256 outputAmount,
        bytes32 slippageProtection,
        address user,
        bool computeOutput
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L30C4-L36C9

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L38C4-L44C9

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L46C5-L52C7


# G-03. Use hardcoded address instead of address(this) #

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this. 

Reference : https://code4rena.com/reports/2023-08-goodentry#g-10-use-hardcoded-address-instead-of-addressthis
```
file : src/ocean
/Ocean.sol

836 :             SafeERC20.safeTransferFrom(IERC20(tokenAddress), userAddress, address(this), transferAmount);

891 :         IERC721(tokenAddress).safeTransferFrom(userAddress, address(this), tokenId);

904 :         IERC721(tokenAddress).safeTransferFrom(address(this), userAddress, tokenId);

929 :         if (tokenAddress == address(this)) revert NO_RECURSIVE_WRAPS();

931 :        IERC1155(tokenAddress).safeTransferFrom(userAddress, address(this), tokenId, amount, "");

964 :         if (tokenAddress == address(this)) revert NO_RECURSIVE_UNWRAPS();

968 :         IERC1155(tokenAddress).safeTransferFrom(address(this), userAddress, tokenId, amountRemaining, "");
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L836

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L891

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L904

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L929

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L931

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L964

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L968
```
file : src/adapters
/CurveTricryptoAdapter.sol

213 :                 uint256 wethBalance = IERC20Metadata(underlying[zToken]).balanceOf(address(this));

216 :                     IERC20Metadata(underlying[zToken]).balanceOf(address(this)) - wethBalance
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L213

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L216

# G-04. Use constants instead of type(uintx).max #

type(uint120).max or type(uint112).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

Reference : https://code4rena.com/reports/2023-08-goodentry#g-20-use-constants-instead-of-typeuintxmax
```
file : src/ocean
/Ocean.sol

102 :     uint256 constant GET_BALANCE_DELTA = type(uint256).max;

170 :         unwrapFeeDivisor = type(uint256).max;
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L102

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L170

# G-05.  OR in if-condition can be rewritten to two single if conditions #

Refactoring the if-condition in a way it won’t be containing the || operator will save more gas

Reference : https://code4rena.com/reports/2023-08-shell#g-05-or-in-if-condition-can-be-rewritten-to-two-single-if-conditions
```
file : src/ocean
/Ocean.sol

709 :         if (interactionType == InteractionType.WrapErc20 || interactionType == InteractionType.UnwrapErc20) {
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L709
```
file : src/adapters
/Curve2PoolAdapter.sol

209 :         if (((inputToken == xToken) && (outputToken == yToken)) || ((inputToken == yToken) && (outputToken == xToken)))

212 :         } else if (((inputToken == xToken) || (inputToken == yToken)) && (outputToken == lpTokenId)) {

214 :         } else if ((inputToken == lpTokenId) && ((outputToken == xToken) || (outputToken == yToken))) {
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L209

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L212

https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L214

# G-06. Using a positive conditional flow to save a NOT opcode #

Estimated savings: 3 gas

Reference : https://code4rena.com/reports/2023-07-basin#g-13-using-a-positive-conditional-flow-to-save-a-not-opcode
```
File : src/ocean
/Ocean.sol

186 :         if (!isApprovedForAll(userAddress, msg.sender)) revert FORWARDER_NOT_APPROVED();
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L186

# G-07. Instead of address(0) write it out, it saves gas #

use : 0x00000000000000000000000
instead of : address(0)
```
file : src/adapters
/CurveTricryptoAdapter.sol

152 :                 interactionTypeAndAddress: _fetchInteractionId(address(0), uint256(InteractionType.UnwrapEther)),
```
https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/CurveTricryptoAdapter.sol#L152