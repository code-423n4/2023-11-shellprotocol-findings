# [L-01] Use of ecrecover is Susceptible to Signature Malleability

The built-in EVM pre-compiled `ecrecover` featured in the function is susceptible to signature malleability due to non-unique v and s (which may not always be in the lower half of the modulo set) values, possibly leading to replay attacks. Elsewhere if devoid of the adoption of nonces, this could prove a vulnerability when not carefully used.

        address recoveredAddress = ecrecover(digest, v, r, s);

https://github.com/code-423n4/2023-11-shellprotocol/blob/f6df0c659d80d215cb0f91a4ea7afcbf695eb390/src/ocean/ERC1155PermitSignatureExtension.sol#L46

## Recommended Mitigation Steps
Consider using OpenZeppelin’s ECDSA library which has been time tested in preventing this malleability where possible.