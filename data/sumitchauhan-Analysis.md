High risk issues
1. Ether sent to arbitrary destinations:
   Ocean._etherUnwrap(uint256,address) (contracts/Ocean.sol#978-984) sends eth to 
   arbitrary user
        Dangerous calls:
        - payable(userAddress).transfer(transferAmount) (contracts/Ocean.sol#982)

2. Reentrancy:
   Reentrancy in Ocean._doInteraction(Interaction,address) 
   (contracts/Ocean.sol#380-430):     
   Reentrancy in Ocean._doMultipleInteractions(Interaction[],uint256[],address) 
   (contracts/Ocean.sol#445-573):
   Reentrancy in 
   Ocean._computeInputAmount(address,uint256,uint256,uint256,address,bytes32) 
   (contracts/Ocean.sol#786-806)
   Reentrancy in 
   Ocean._computeOutputAmount(address,uint256,uint256,uint256,address,bytes32) 
   (contracts/Ocean.sol#745-765)
   Reentrancy in Ocean._erc1155Wrap(address,uint256,uint256,address,uint256) 
   (contracts/Ocean.sol#920-934)
   Reentrancy in Ocean._erc721Wrap(address,uint256,address,uint256) 
   (contracts/Ocean.sol#889-894)
   Reentrancy in 
   Curve2PoolAdapter.primitiveOutputAmount(uint256,uint256,uint256,bytes32) 
   (contracts/Curve2PoolAdapter.sol#142-184):
   Reentrancy in 
   CurveTricryptoAdapter.primitiveOutputAmount(uint256,uint256,uint256,bytes32) 
   (contracts/CurveTricryptoAdapter.sol#178-236):
       
Medium risk issues
1. Unused return:
Curve2PoolAdapter.wrapToken(uint256,uint256) (contracts/Curve2PoolAdapter.sol#102-114) ignores return value by IOceanInteractions(ocean).doInteraction(interaction)
Curve2PoolAdapter.unwrapToken(uint256,uint256) (contracts/Curve2PoolAdapter.sol#121-133) ignores return value by IOceanInteractions(ocean).doInteraction(interaction) 
Curve2PoolAdapter._approveToken(address) (contracts/Curve2PoolAdapter.sol#189-192) ignores return value by IERC20Metadata(tokenAddress).approve(ocean,type()(uint256).max) 
Curve2PoolAdapter._approveToken(address) (contracts/Curve2PoolAdapter.sol#189-192) ignores return value by IERC20Metadata(tokenAddress).approve(primitive,type()(uint256).max) 


Gas optimizations or Informational issues
1. Usage of `uint`/`int` smaller than 32 bytes (256 bits) incurs overhead:
  Ocean.sol::99 => uint8 constant NORMALIZED_DECIMALS = 18;
  Ocean.sol::688 => interactionType = 
  InteractionType(uint8(interactionTypeAndAddress[0]));
  Ocean.sol::821 => try IERC20Metadata(tokenAddress).decimals() returns (uint8 
  decimals) {
  Ocean.sol::865 => try IERC20Metadata(tokenAddress).decimals() returns (uint8 
  decimals) {
  Ocean.sol::821 => uint8 decimals
  Ocean.sol::1124 => uint8 decimalsFrom,
  Ocean.sol::1125 => uint8 decimalsTo,
  OceanAdapter.sol::16 => uint8 constant NORMALIZED_DECIMALS = 18;
  OceanAdapter.sol::139 => uint8 decimalsFrom,
  OceanAdapter.sol::140 => uint8 decimalsTo,
  CurveTricryptoAdapter.sol::76 => mapping(uint256 => uint8) decimals;
  Curve2PoolAdapter.sol::68 => mapping(uint256 => uint8) decimals;
  Curve2PoolAdapter.sol::86 => indexOf[yToken] = int128(1);
  Curve2PoolAdapter.sol::159 => int128 indexOfInputAmount = indexOf[inputToken];
  Curve2PoolAdapter.sol::160 => int128 indexOfOutputAmount = 
  indexOf[outputToken];
  
2. Constants in comparisons should appear on the left side:
  Ocean.sol::391 => if (msg.value != 0) {
  Ocean.sol::419 => if (inputAmount > 0) {
  Ocean.sol::426 => if (outputAmount > 0) {
  Ocean.sol::474 => if (msg.value != 0) {
  Ocean.sol::527 => if (inputAmount > 0) {
  Ocean.sol::533 => if (outputAmount > 0) {
  Ocean.sol::554 => if (mintIds.length == 1) {
  Ocean.sol::558    => } else if (mintIds.length > 1) {
  Ocean.sol::564 => if (burnIds.length == 1) {
  Ocean.sol::568 => } else if (burnIds.length > 1) {
  Ocean.sol::640 => if (specifiedAmount != 1) revert INVALID_ERC721_AMOUNT();
  Ocean.sol::648 => if (specifiedAmount != 1) revert INVALID_ERC721_AMOUNT();
  Ocean.sol::929 => if (tokenAddress == address(this)) revert 
  NO_RECURSIVE_WRAPS();
  Ocean.sol::964 => if (tokenAddress == address(this)) revert 
  NO_RECURSIVE_UNWRAPS();
  Ocean.sol::1009 => if (_isNotTokenOfPrimitive(inputToken, primitive) && 
  (inputAmount > 0)) {
  Ocean.sol::1043 => if (_isNotTokenOfPrimitive(outputToken, primitive) && 
  (outputAmount > 0)) {
  Ocean.sol::1084 => if (truncated > 0) {
  Ocean.sol::1167 => if (amount > 0) {
  
3. Use multiple `require()` and `if` statements instead of `&&`:
   Ocean.sol::1009 => if (_isNotTokenOfPrimitive(inputToken, primitive) && 
   (inputAmount > 0)) {
   Ocean.sol::1043 => if (_isNotTokenOfPrimitive(outputToken, primitive) && 
   (outputAmount > 0)) {
   Curve2PoolAdapter.sol::209 => if (((inputToken == xToken) && (outputToken == 
   yToken)) || ((inputToken == yToken) && (outputToken == xToken)))
   Curve2PoolAdapter.sol::212 => } else if (((inputToken == xToken) || 
   (inputToken == yToken)) && (outputToken == lpTokenId)) {
   Curve2PoolAdapter.sol::214 => } else if ((inputToken == lpTokenId) && 
   ((outputToken == xToken) || (outputToken == yToken))) {
  
4. Function state mutability can be restricted to pure:
   OceanAdapter.sol:124 => function getTokenSupply(uint256 tokenId) external view 
   override returns (uint256) {
   
5. Incorrect versions of solidity:
   Pragma version0.8.20 (contracts/OceanAdapter.sol#4) necessitates a version too 
   recent to be trusted. solc-0.8.20 is not recommended for deployment. Consider 
   deploying with 0.8.18.
   
6. Unused state variable:
   OceanAdapter.NORMALIZED_DECIMALS (contracts/OceanAdapter.sol#16) is never used 
   in OceanAdapter 
   
   

### Time spent:
16 hours