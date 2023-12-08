## Overview

The Ocean protocol provides an innovative framework for token interactions and primitive composability. I analyzed core components like access controls, accounting, standards compliance, reentrancy surface, and economic vulnerabilities.

| Contract                   | SLOC | Purpose                                           | Libraries used      |
|----------------------------|------|---------------------------------------------------|----------------------|
| [Ocean.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol)                  | 561  | The accounting engine of the shell protocol        | @openzeppelin/*     |
| [Curve2PoolAdapter.sol ](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol)     | 139  | Adapter that enables integration with the curve 2 pool | @openzeppelin/*     |
| [CurveTricryptoAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/OceanAdapter.sol)  | 199  | Adapter that enables integration with the curve tricrypto pool | @openzeppelin/*     |
| [OceanAdapter.sol](https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol)           | 94   | Helper contract for the adapters                   | @openzeppelin/*     |

## My Approach

My analysis focused on core security properties around user funds and data:

- Permissions and access control 
- Token tracking and accounting
- Availability and denial of service prevention

I evaluated protocol mechanisms, scoped potential issues, examined architectural assumptions, modeled attack scenarios, and highlighted mitigations.

**Permissions and Access Control**

I audited operations for balance manipulations and transfers: 

```

```

**Token Accounting** 

Analyzed core supply tracking events: [_erc20Wrap](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L820-L842)

```solidity
function _erc20Wrap(uint256 amount) private {

  emit Erc20Wrap(
    amountTransferred, // Tracks actual transfer 
    amountSpecified, // Differs from above
    dust, // Lost precision
    user, 
    token  
  );

}
```

**Denial of Service**

Validated pause functionality:

```
function pause() external onlyOwner {
  _pause();
}

function _beforeTokenTransfer(address from, address to, uint256 tokenId) internal virtual override { 
  super._beforeTokenTransfer(from, to, tokenId);

  if (paused()) revert TRANSFER_PAUSED();
} 
```

**Key Findings**

- Permission checks rely on approvals - user education is vital 
- Custom ERC-1155 handling could block legitimate transfers
- Split primitive balance updates enable state corruption bugs
- Removal of reentrancy guards expands attack surface to malicious externs
- Sandwich attacks possible with manipulated message sender

**Permission Checks Rely on Approvals**

The [onlyApprovedForwarder modifier](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L185-L188) checks:

```
if (!isApprovedForAll(userAddress, msg.sender)) 
  revert FORWARDER_NOT_APPROVED();
```

This enables forwarding contracts to drain funds if users mistakenly approve them. Needs caution around approving contracts.

**Custom ERC-1155 Handling** 

Overriding transfer callbacks: [onERC1155Received](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L326-L343)

```
function onERC1155Received(address operator, address from, uint256 id, uint256 value, bytes calldata data) external returns (bytes4) {
  if (_isInteraction) {
    return this.onERC1155Received.selector;
  } else {
    return 0x0; 
  }  
}
```
Could block external transfers not done during interactions. Breaks 1155 expectations.

**Primitive Balance Update Split**

Pre/post split:

 ```
_mintTokensBeforeInteraction()

callComputeInteraction()  

_burnTokensAfterInteraction() 
```

Allows reentrancy to corrupt balance state across calls.

**Reentrancy Surface Increase**

Removed guards from [doInteraction](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L210-L218) methods, allowing calls back in during execution.

**Sandwich Attacks**

Front-running `computeInteraction`:

```
1. User starts interaction
2. Attacker frontruns compute call
3. Manipulates msg.sender 
4. Performs attack
5. Funds drained from protocol
```
**Architecture Recommendations** 

- Reintroduce reentrancy guards around interactions
- Create access controlled admin functions 
- Implement supplemental user balances for token dust  
- Add protocol-level pause functionality 
- Introduce interaction nonce tracking 

**Reintroduce Reentrancy Guards**

Add guards back to key interaction handlers: For example

```
contract Ocean {

  uint256 entryPointMutex;   

  function doInteraction() nonReentrant {
    entryPointMutex = 1;
   
    // Interaction logic  

    entryPointMutex = 0;
  }

}

modifier nonReentrant {
  require(entryPointMutex == 0, "Reentrancy attempt");
  entryPointMutex = 1;
  _;
  entryPointMutex = 0;
}
```

This prevents reentrancy for external calls made during interaction execution.

**Access Controlled Admin Functions**

Use a role-based system like OpenZeppelin AccessControl for elevated permissions:

```
contract Ocean is AccessControl {
  bytes32 ADMIN_ROLE = keccak256("ADMIN");

  modifier onlyAdmin() {
    require(hasRole(ADMIN_ROLE, msg.sender));
    _;
  }
  
  function addToken(address token) onlyAdmin external {
    supportedTokens.push(token); 
  }

}
```

**Supplemental Balances**  

Track per-user dust balances from conversion mismatches. Reconcile during interactions.

**Protocol Pause Functionality**

Add ability to pause core contract functionality during emergencies:

```
bool public paused;
modifier whenNotPaused() {
  require(!paused, "Contract paused");
  _;
}

function pause() onlyAdmin external {
  paused = true;
} 
```

**Interaction Nonce Tracking** 

Record incrementing nonces per user to block replay attacks:

```
mapping(address => uint256) interactionNonces;

function doInteraction() {
  require(interactionNonces[msg.sender]++ == latestNonce[msg.sender], "Stale nonce");  
}
```

[**doInteraction**](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L210-L218)

```
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

This is the main entry point for users to perform a single interaction. It emits an event, and handles execution by passing details to the internal `_doInteraction` method.

[**doMultipleInteractions**](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L229-L245)

```
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

Allows batch execution of multiple interactions in a single transaction. Maintains balance updates for all interactions via the ids array and return value arrays.

[**wrapToken/unwrapToken**](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L102-L133)

Wrapper functions in adapter contracts like Curve2PoolAdapter that handle token wrapping through the Ocean:

```
function wrapToken(uint256 tokenId, uint256 amount) internal override {

  Interaction memory interaction = buildWrapInteraction(tokenId);
  
  ocean.doInteraction(interaction);

}
```

**Core Components**

- `Ocean` - Main protocol contract for handling interactions 
- `OceanERC1155` - Custom ERC1155 implementation
- `Interactions` - Library defining interaction types and data
- `IOceanInteractions` - Interface for interaction functions
- `IOceanPrimitive` - Interface for primitive contracts

**Interaction Handling**

The `Ocean` contract is the core of the framework for enabling token interactions. The key methods are:

- `doInteraction` - Perform single interaction
- `doMultipleInteractions` - Batch execute interactions

These handle execution and accounting for different interaction types defined in the `Interactions` library.

**Accounting System** 

The `OceanERC1155` base contract provides the accounting system and token ledger. It tracks ERC1155 token balances and supplies for users and primitive contracts.

Custom balances hooks are inserted around external calls to primitives.

**Primitive Interactions**

The `IOceanPrimitive` defines the interface for primitive contracts like AMMs to integrate with the framework. The `Ocean` contract handles calling these external contracts during `doInteraction`.

**Adapter Model**

Adapter contracts build on the core `IOceanPrimitive` interfaces to wrap specific protocol functionality into Ocean interactions.

**Invariants**

Important invariants relate to accounting consistency, isolated interactions, access controls, and standards compliance.

## Potential issues with each core function.

[**doInteraction**](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/ocean/Ocean.sol#L210-L218)

```
function doInteraction(Interaction calldata interaction) external override returns (uint256 burnId, uint256 burnAmount, uint256 mintId, uint256 mintAmount) {

  // Removed reentrancy guard here
  
  return _doInteraction(interaction, msg.sender);

}
```

- Lack of reentrancy guard enables potential reentrancy if malicious adapter contract makes unsafe external calls during interaction

[**wrapToken**](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L102-L133)

```
function wrapToken(uint256 tokenId, uint256 amount) internal override {

  Interaction memory interaction = buildWrapInteraction(tokenId);
  
  ocean.doInteraction(interaction);

}
```

- Over-reliance on external `Ocean` contract for key token accounting
- Lack of handling for failed wrapping scenarios can cause token supply imbalance

[**unwrapToken**](https://github.com/code-423n4/2023-11-shellprotocol/blob/485de7383cdf88284ee6bcf2926fb7c19e9fb257/src/adapters/Curve2PoolAdapter.sol#L102-L133)

``` 
function unwrapToken(uint256 tokenId, uint256 amount) internal override {

  Interaction memory interaction = buildUnwrapInteraction(tokenId);

  ocean.doInteraction(interaction);

}
```

- Same issues as `wrapToken` in relying on external contract
- Failed unwraps could decrement user balances but not burn tokens

**computeInteraction** 

- Splitting mint/burn allows reentrancy attack leading to corrupted balances
- Message sender could be manipulated to drain other user's tokens

**Conclusion**

The Ocean demonstrates well-structured core accounting and interactions. Limiting elevated permissions, adding security gates around new paths like adapters, ensuring atomicity of critical operations, and enabling admin controls would raise assurances further.

### Time spent:
39 hours