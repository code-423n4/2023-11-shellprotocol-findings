## Wrong balance update info
The `_grantFeeToOcean` calls `_mintWithoutSafeTransferAcceptanceCheck` which updates the balance of tokens for the owner of the contract. However, it is known that for every wrapping, the tokens are sent to the Ocean contract instead of the owner. Thus, in context,  it should be `_mintWithoutSafeTransferAcceptanceCheck(address(this)), oceanId, amount);` instead of `_mintWithoutSafeTransferAcceptanceCheck(owner(), oceanId, amount); `

## Missing Fallback/Receive Function
Ocean.sol interacts with ether, and it is good practice to have a Fallback or Receive function.
