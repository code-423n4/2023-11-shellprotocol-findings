## [L-1] `_setApprovalForAll.owner` function parameter shadows `Ownable.owner`.

The `owner` parameter of the `_setApprovalForAll` function shadows the inherited `Ownable.owner` state variable of OceanERC1155.sol contract.
See: https://swcregistry.io/docs/SWC-119/

File: https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/OceanERC1155.sol#L532
OceanERC1155.sol#_setApprovalForAll().owner
```
function _setApprovalForAll(address owner, address operator, bool approved) internal override {
        require(owner != operator, "Set approval for self");
        _operatorApprovals[owner][operator] = approved;
        emit ApprovalForAll(owner, operator, approved);
    }
```
Ocean inherits OceanERC1155
```
contract Ocean is IOceanInteractions, IOceanFeeChange, OceanERC1155, IERC721Receiver, IERC1155Receiver {
```
OceanERC1155.sol inherits `Ownable.sol` with `owner` state variable
```
contract OceanERC1155 is
    Context,
    ERC165,
    ERC1155PermitSignatureExtension,
    IERC1155,
    IERC1155MetadataURI,
    IOceanToken,
    Ownable,
    ReentrancyGuard
{
```
Recommendation: rename the `owner` parameter of `OceanERC1155.sol#_setApprovalForAll` function to `_owner`.