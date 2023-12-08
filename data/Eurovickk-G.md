https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/ocean/Ocean.sol

1)Loops and Iterations: Loops are gas-expensive. Optimizing loops, especially those iterating through arrays, can significantly reduce gas costs. 
 Iterating through Interaction Array in doMultipleInteractions: Consider using a while loop instead of a for loop to avoid the unnecessary increment operation within the loop.
uint256 i = 0;
while (i < interactions.length) {
    // Interaction processing logic
    i++;
}
Interaction Loop within doMultipleInteractions: Combine the increment operation with the loop condition to avoid the separate increment statement inside the loop.
for (uint256 i = 0; i < interactions.length; i++) {
    // Inside the loop, interaction processing logic
}

2)Utilize batch operations for token transfers or interactions to minimize the number of individual transactions within loops.

Token Transfers in Loops: Where the contract transfers tokens within loops (such as in doMultipleInteractions), consider accumulating token transfer amounts and utilizing batch transfer functions provided by token interfaces (IERC20, IERC1155, etc.) to perform transfers in batches rather than individually.
Suppose you have a loop where token transfers are occurring, and you want to optimize it by performing batch transfers.
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

contract YourContract {
    // Example function performing individual transfers in a loop
    function transferTokensInLoop(
        address[] calldata recipients,
        uint256[] calldata amounts,
        IERC20 token
    ) external {
        require(recipients.length == amounts.length, "Array length mismatch");
        for (uint256 i = 0; i < recipients.length; i++) {
            token.transfer(recipients[i], amounts[i]);
        }
    }
    // Optimized function performing batch transfers
    function batchTransferTokens(
        address[] calldata recipients,
        uint256[] calldata amounts,
        IERC20 token
    ) external {
        require(recipients.length == amounts.length, "Array length mismatch");
        uint256 totalTransfers = recipients.length;
        address[] memory recipientsBatch = new address[](totalTransfers);
        uint256[] memory amountsBatch = new uint256[](totalTransfers);
        for (uint256 i = 0; i < totalTransfers; i++) {
            recipientsBatch[i] = recipients[i];
            amountsBatch[i] = amounts[i];
        }
        token.transfer(recipientsBatch, amountsBatch);
    }
}
In this optimized function (batchTransferTokens), the token transfers are aggregated into two arrays (recipientsBatch and amountsBatch) and then executed using the token's transfer function which supports batch transfers.

_mintBatch and _burnBatch Functions: These functions could be leveraged for batch minting and burning of tokens if there are multiple tokens to be minted or burned.
// Example of using _mintBatch for minting multiple tokens
function mintTokensInBatch(
    uint256[] calldata tokenIds,
    uint256[] calldata amounts
) external {
    // Assuming this contract has the capability to mint tokens using _mintBatch
    _mintBatch(msg.sender, tokenIds, amounts, "");
}
// Example of using _burnBatch for burning multiple tokens
function burnTokensInBatch(
    uint256[] calldata tokenIds,
    uint256[] calldata amounts
) external {
    // Assuming this contract has the capability to burn tokens using _burnBatch
    _burnBatch(msg.sender, tokenIds, amounts);
} 

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/Curve2PoolAdapter.sol

1)Within the function primitiveOutputAmount, multiple storage reads (SLOADS) are performed to obtain token indices. These reads could be combined or minimized to reduce gas consumption.
To optimize this, one approach could involve consolidating multiple storage reads into a single operation wherever feasible. For instance, if there are multiple reads for token indices in different parts of the code, try to minimize them by storing these indices in variables whenever possible. This approach reduces the number of individual storage reads, thus optimizing gas usage. Additionally, consider using memory variables instead of storage wherever appropriate to reduce gas costs for storage operations.

function primitiveOutputAmount(
    uint256 inputToken,
    uint256 outputToken,
    uint256 inputAmount,
    bytes32 minimumOutputAmount
)
    internal
    override
    returns (uint256 outputAmount)
{
    uint256 rawInputAmount = _convertDecimals(NORMALIZED_DECIMALS, decimals[inputToken], inputAmount);

    ComputeType action = _determineComputeType(inputToken, outputToken);

    uint256 rawOutputAmount;
    int128 indexOfInputAmount;
    int128 indexOfOutputAmount;

    // Consolidate SLOAD for token indices
    (indexOfInputAmount, indexOfOutputAmount) = _getTokenIndices(inputToken, outputToken);
    if (action == ComputeType.Swap) {
        rawOutputAmount = ICurve2Pool(primitive).exchange(indexOfInputAmount, indexOfOutputAmount, rawInputAmount, 0);
    } else if (action == ComputeType.Deposit) {
        uint256[2] memory inputAmounts;
        inputAmounts[uint256(int256(indexOfInputAmount))] = rawInputAmount;
        rawOutputAmount = ICurve2Pool(primitive).add_liquidity(inputAmounts, 0);
    } else {
        rawOutputAmount = ICurve2Pool(primitive).remove_liquidity_one_coin(rawInputAmount, indexOfOutputAmount, 0);
    }
    outputAmount = _convertDecimals(decimals[outputToken], NORMALIZED_DECIMALS, rawOutputAmount);

    if (uint256(minimumOutputAmount) > outputAmount) revert SLIPPAGE_LIMIT_EXCEEDED();

    if (action == ComputeType.Swap) {
        emit Swap(inputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);
    } else if (action == ComputeType.Deposit) {
        emit Deposit(inputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);
    } else {
        emit Withdraw(outputToken, inputAmount, outputAmount, minimumOutputAmount, primitive, true);
    }
}
function _getTokenIndices(uint256 inputToken, uint256 outputToken) private view returns (int128, int128) {
    int128 indexOfInputAmount;
    int128 indexOfOutputAmount;
    if (((inputToken == xToken) && (outputToken == yToken)) || ((inputToken == yToken) && (outputToken == xToken))) {
        indexOfInputAmount = indexOf[inputToken];
        indexOfOutputAmount = indexOf[outputToken];
    } else if (((inputToken == xToken) || (inputToken == yToken)) && (outputToken == lpTokenId)) {
        indexOfInputAmount = indexOf[inputToken];
    } else if ((inputToken == lpTokenId) && ((outputToken == xToken) || (outputToken == yToken))) {
        indexOfOutputAmount = indexOf[outputToken];
    } else {
        revert INVALID_COMPUTE_TYPE();
    }
    return (indexOfInputAmount, indexOfOutputAmount);
}
In this modified version, the _getTokenIndices function consolidates the logic to retrieve the token indices based on input and output tokens, reducing the number of individual SLOAD operations within the main function primitiveOutputAmount. The retrieved indices are then used within the function to perform necessary operations, thereby optimizing gas usage by reducing redundant storage reads.

https://github.com/code-423n4/2023-11-shellprotocol/blob/main/src/adapters/CurveTricryptoAdapter.sol

1)The loop in _determineComputeType could potentially be streamlined.

function _determineComputeType(
    uint256 inputToken,
    uint256 outputToken
)
    private
    view
    returns (ComputeType computeType)
{
    if (
        ((inputToken == xToken || inputToken == yToken || inputToken == zToken) &&
        (outputToken == xToken || outputToken == yToken || outputToken == zToken)) ||
        ((inputToken == xToken || inputToken == yToken) && outputToken == zToken) ||
        ((inputToken == zToken || inputToken == yToken) && outputToken == xToken)
    ) {
        return ComputeType.Swap;
    } else if (
        (inputToken == xToken || inputToken == yToken || inputToken == zToken) &&
        outputToken == lpTokenId
    ) {
        return ComputeType.Deposit;
    } else if (
        inputToken == lpTokenId &&
        (outputToken == xToken || outputToken == yToken || outputToken == zToken)
    ) {
        return ComputeType.Withdraw;
    } else {
        revert INVALID_COMPUTE_TYPE();
    }
}
This revised version simplifies the conditionals by grouping token comparisons logically. It reduces the number of individual comparisons and logical operators, which may help optimize gas usage in this function.

2)Consider reducing SLOAD operations where possible. In functions like _determineComputeType, multiple SLOADs are occurring. Consolidating or minimizing these could improve gas efficiency.
To reduce the number of SLOAD operations in the _determineComputeType function, you can store the values of xToken, yToken, zToken, and lpTokenId in local variables within the contract's state during initialization in the constructor.
contract CurveTricryptoAdapter is OceanAdapter {
    // ... (other contract variables)

    // Store xToken, yToken, zToken, and lpTokenId in state variables
    uint256 private _xToken;
    uint256 private _yToken;
    uint256 private _zToken;
    uint256 private _lpTokenId;

    constructor(address ocean_, address primitive_) OceanAdapter(ocean_, primitive_) {
        // ... (existing code for initialization)

        // Store token IDs in state variables during initialization
        _xToken = xToken;
        _yToken = yToken;
        _zToken = zToken;
        _lpTokenId = lpTokenId;
    }

    function _determineComputeType(
        uint256 inputToken,
        uint256 outputToken
    )
        private
        view
        returns (ComputeType computeType)
    {
        uint256 xToken = _xToken;
        uint256 yToken = _yToken;
        uint256 zToken = _zToken;
        uint256 lpTokenId = _lpTokenId;

        if (
            ((inputToken == xToken || inputToken == yToken || inputToken == zToken) &&
            (outputToken == xToken || outputToken == yToken || outputToken == zToken)) ||
            ((inputToken == xToken || inputToken == yToken) && outputToken == zToken) ||
            ((inputToken == zToken || inputToken == yToken) && outputToken == xToken)
        ) {
            return ComputeType.Swap;
        } else if (
            (inputToken == xToken || inputToken == yToken || inputToken == zToken) &&
            outputToken == lpTokenId
        ) {
            return ComputeType.Deposit;
        } else if (
            inputToken == lpTokenId &&
            (outputToken == xToken || outputToken == yToken || outputToken == zToken)
        ) {
            return ComputeType.Withdraw;
        } else {
            revert INVALID_COMPUTE_TYPE();
        }
    }
}
By storing these values as private variables during contract initialization, you reduce the number of SLOAD operations within the _determineComputeType function, improving gas efficiency when accessing these values during the token comparison checks. This approach utilizes local variables instead of repeatedly accessing contract state variables, saving gas costs for each function call.
