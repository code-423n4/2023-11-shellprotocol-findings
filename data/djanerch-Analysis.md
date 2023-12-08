# Medium Issues

## M-1: Centralization Risk for trusted owners

Contracts have owners with privileged rights to perform admin tasks and need to be trusted to not perform malicious updates or drain funds.

- Found in src/ocean/Ocean.sol: Line: 196
- Found in src/ocean/OceanERC1155.sol: Line: 36
- Found in src/ocean/OceanERC1155.sol: Line: 84
- Found in src/proteus/LiquidityPoolProxy.sol: Line: 10
- Found in src/proteus/LiquidityPoolProxy.sol: Line: 42
- Found in src/proteus/LiquidityPoolProxy.sol: Line: 47


## M-2: Using `ERC721::_mint()` can be dangerous

Using `ERC721::_mint()` can mint ERC721 tokens to addresses which don't support ERC721 tokens. Use `_safeMint()` instead of `_mint()` for ERC721.

- Found in src/mock/ERC1155MintsToDeployer.sol: Line: 10
- Found in src/mock/ERC20MintsToDeployer.sol: Line: 12
- Found in src/ocean/Ocean.sol: Line: 428
- Found in src/ocean/Ocean.sol: Line: 557


# Low Issues

## L-1: `ecrecover` is susceptible to signature malleability

The `ecrecover` function is susceptible to signature malleability. This means that the same message can be signed in multiple ways, allowing an attacker to change the message signature without invalidating it. This can lead to unexpected behavior in smart contracts, such as the loss of funds or the ability to bypass access control. Consider using OpenZeppelin's ECDSA library instead of the built-in function.

- Found in src/ocean/ERC1155PermitSignatureExtension.sol: Line: 46


## L-2: Unsafe ERC20 Operations should not be used

ERC20 functions may not behave as expected. For example: return values are not always meaningful. It is recommended to use OpenZeppelin's SafeERC20 library.

- Found in src/adapters/Curve2PoolAdapter.sol: Line: 190
- Found in src/adapters/Curve2PoolAdapter.sol: Line: 191
- Found in src/adapters/CurveTricryptoAdapter.sol: Line: 242
- Found in src/adapters/CurveTricryptoAdapter.sol: Line: 243
- Found in src/ocean/Ocean.sol: Line: 982


## L-3: Conditional storage checks are not consistent

When writing `require` or `if` conditionals that check storage values, it is important to be consistent to prevent off-by-one errors. There are instances found where the same storage variable is checked multiple times, but the conditionals are not consistent.

- Found in src/proteus/EvolvingProteus.sol: Line: 241
- Found in src/proteus/EvolvingProteus.sol: Line: 584
- Found in src/proteus/EvolvingProteus.sol: Line: 644
- Found in src/proteus/EvolvingProteus.sol: Line: 706
- Found in src/proteus/EvolvingProteus.sol: Line: 742
- Found in src/proteus/EvolvingProteus.sol: Line: 793
- Found in src/proteus/EvolvingProteus.sol: Line: 813
- Found in src/proteus/EvolvingProteus.sol: Line: 817
- Found in src/proteus/EvolvingProteus.sol: Line: 820
- Found in src/proteus/LiquidityPool.sol: Line: 141
- Found in src/proteus/LiquidityPool.sol: Line: 176
- Found in src/proteus/LiquidityPool.sol: Line: 180
- Found in src/proteus/LiquidityPool.sol: Line: 279
- Found in src/proteus/LiquidityPool.sol: Line: 405
- Found in src/proteus/LiquidityPool.sol: Line: 409
- Found in src/proteus/LiquidityPool.sol: Line: 416
- Found in src/proteus/LiquidityPool.sol: Line: 426
- Found in src/proteus/LiquidityPool.sol: Line: 452
- Found in src/proteus/LiquidityPool.sol: Line: 455
- Found in src/proteus/Proteus.sol: Line: 388
- Found in src/proteus/Proteus.sol: Line: 576
- Found in src/proteus/Proteus.sol: Line: 632
- Found in src/proteus/Proteus.sol: Line: 639
- Found in src/proteus/Proteus.sol: Line: 662
- Found in src/proteus/Proteus.sol: Line: 668
- Found in src/proteus/Proteus.sol: Line: 690
- Found in src/proteus/Proteus.sol: Line: 694
- Found in src/proteus/Proteus.sol: Line: 715
- Found in src/proteus/Proteus.sol: Line: 719
- Found in src/proteus/Proteus.sol: Line: 722
- Found in src/proteus/Slices.sol: Line: 8
- Found in src/proteus/Slices.sol: Line: 30
- Found in src/proteus/Slices.sol: Line: 31
- Found in src/proteus/Slices.sol: Line: 32
- Found in src/proteus/Slices.sol: Line: 33


# NC Issues

## NC-1: Functions not used internally could be marked external



- Found in src/adapters/OceanAdapter.sol: Line: 117
- Found in src/mock/ConstantSum.sol: Line: 22
- Found in src/mock/ConstantSum.sol: Line: 30
- Found in src/mock/ConstantSum.sol: Line: 42
- Found in src/mock/ConstantSum.sol: Line: 54
- Found in src/mock/ConstantSum.sol: Line: 58
- Found in src/mock/ConstantSum.sol: Line: 70
- Found in src/mock/ERC20MintsToDeployer.sol: Line: 16
- Found in src/mock/MaliciousPrimitive.sol: Line: 58
- Found in src/mock/MockPrimitive.sol: Line: 101
- Found in src/mock/Receive1155.sol: Line: 9
- Found in src/mock/RecursiveMaliciousPrimitive.sol: Line: 112
- Found in src/ocean/Ocean.sol: Line: 305
- Found in src/ocean/OceanERC1155.sol: Line: 92
- Found in src/ocean/OceanERC1155.sol: Line: 107
- Found in src/ocean/OceanERC1155.sol: Line: 130
- Found in src/ocean/OceanERC1155.sol: Line: 154
- Found in src/ocean/OceanERC1155.sol: Line: 168
- Found in src/ocean/OceanERC1155.sol: Line: 187
- Found in src/proteus/EvolvingProteus.sol: Line: 233
- Found in src/proteus/LiquidityPoolProxy.sol: Line: 56
- Found in src/proteus/LiquidityPoolProxy.sol: Line: 83
- Found in src/proteus/LiquidityPoolProxy.sol: Line: 112
- Found in src/proteus/LiquidityPoolProxy.sol: Line: 133
- Found in src/proteus/LiquidityPoolProxy.sol: Line: 160
- Found in src/proteus/LiquidityPoolProxy.sol: Line: 181


## NC-2: Constants should be defined and used instead of literals



- Found in src/adapters/Curve2PoolAdapter.sol: Line: 84
- Found in src/adapters/Curve2PoolAdapter.sol: Line: 86
- Found in src/adapters/Curve2PoolAdapter.sol: Line: 166
- Found in src/adapters/CurveTricryptoAdapter.sol: Line: 92
- Found in src/adapters/CurveTricryptoAdapter.sol: Line: 94
- Found in src/adapters/CurveTricryptoAdapter.sol: Line: 99
- Found in src/adapters/CurveTricryptoAdapter.sol: Line: 100
- Found in src/adapters/CurveTricryptoAdapter.sol: Line: 101
- Found in src/adapters/CurveTricryptoAdapter.sol: Line: 205
- Found in src/adapters/ICurve2Pool.sol: Line: 9
- Found in src/adapters/ICurveTricrypto.sol: Line: 9
- Found in src/adapters/OceanAdapter.sol: Line: 101
- Found in src/adapters/OceanAdapter.sol: Line: 152
- Found in src/adapters/OceanAdapter.sol: Line: 156
- Found in src/mock/MaliciousPrimitive.sol: Line: 34
- Found in src/mock/MaliciousPrimitive.sol: Line: 50
- Found in src/mock/MockPrimitive.sol: Line: 28
- Found in src/mock/MockPrimitive.sol: Line: 45
- Found in src/mock/MockPrimitive.sol: Line: 54
- Found in src/mock/MockPrimitive.sol: Line: 56
- Found in src/mock/MockPrimitive.sol: Line: 77
- Found in src/mock/MockPrimitive.sol: Line: 86
- Found in src/mock/MockPrimitive.sol: Line: 88
- Found in src/mock/Receive1155.sol: Line: 27
- Found in src/mock/Receive1155.sol: Line: 48
- Found in src/mock/RecursiveMaliciousPrimitive.sol: Line: 30
- Found in src/mock/RecursiveMaliciousPrimitive.sol: Line: 50
- Found in src/mock/RecursiveMaliciousPrimitive.sol: Line: 59
- Found in src/mock/RecursiveMaliciousPrimitive.sol: Line: 61
- Found in src/mock/RecursiveMaliciousPrimitive.sol: Line: 63
- Found in src/mock/RecursiveMaliciousPrimitive.sol: Line: 66
- Found in src/mock/RecursiveMaliciousPrimitive.sol: Line: 86
- Found in src/mock/RecursiveMaliciousPrimitive.sol: Line: 95
- Found in src/mock/RecursiveMaliciousPrimitive.sol: Line: 97
- Found in src/mock/RecursiveMaliciousPrimitive.sol: Line: 99
- Found in src/mock/RecursiveMaliciousPrimitive.sol: Line: 103
- Found in src/ocean/Ocean.sol: Line: 173
- Found in src/ocean/Ocean.sol: Line: 216
- Found in src/ocean/Ocean.sol: Line: 266
- Found in src/ocean/Ocean.sol: Line: 554
- Found in src/ocean/Ocean.sol: Line: 558
- Found in src/ocean/Ocean.sol: Line: 564
- Found in src/ocean/Ocean.sol: Line: 568
- Found in src/ocean/Ocean.sol: Line: 640
- Found in src/ocean/Ocean.sol: Line: 648
- Found in src/ocean/Ocean.sol: Line: 1092
- Found in src/ocean/Ocean.sol: Line: 1138
- Found in src/ocean/Ocean.sol: Line: 1143
- Found in src/proteus/EvolvingProteus.sol: Line: 214
- Found in src/proteus/EvolvingProteus.sol: Line: 217
- Found in src/proteus/EvolvingProteus.sol: Line: 695
- Found in src/proteus/EvolvingProteus.sol: Line: 702
- Found in src/proteus/EvolvingProteus.sol: Line: 813
- Found in src/proteus/LiquidityPool.sol: Line: 126
- Found in src/proteus/LiquidityPool.sol: Line: 364
- Found in src/proteus/LiquidityPool.sol: Line: 365
- Found in src/proteus/LiquidityPool.sol: Line: 367
- Found in src/proteus/LiquidityPool.sol: Line: 369
- Found in src/proteus/LiquidityPool.sol: Line: 372
- Found in src/proteus/LiquidityPool.sol: Line: 398
- Found in src/proteus/LiquidityPool.sol: Line: 413
- Found in src/proteus/LiquidityPool.sol: Line: 417
- Found in src/proteus/LiquidityPool.sol: Line: 421
- Found in src/proteus/LiquidityPool.sol: Line: 427
- Found in src/proteus/LiquidityPool.sol: Line: 431
- Found in src/proteus/LiquidityPoolProxy.sol: Line: 39
- Found in src/proteus/Proteus.sol: Line: 51
- Found in src/proteus/Proteus.sol: Line: 251
- Found in src/proteus/Proteus.sol: Line: 256
- Found in src/proteus/Proteus.sol: Line: 357
- Found in src/proteus/Proteus.sol: Line: 360
- Found in src/proteus/Proteus.sol: Line: 471
- Found in src/proteus/Proteus.sol: Line: 473
- Found in src/proteus/Proteus.sol: Line: 475
- Found in src/proteus/Proteus.sol: Line: 476
- Found in src/proteus/Proteus.sol: Line: 477
- Found in src/proteus/Proteus.sol: Line: 517
- Found in src/proteus/Proteus.sol: Line: 548
- Found in src/proteus/Proteus.sol: Line: 641
- Found in src/proteus/Proteus.sol: Line: 644
- Found in src/proteus/Proteus.sol: Line: 658
- Found in src/proteus/Proteus.sol: Line: 715
- Found in src/proteus/Slices.sol: Line: 36
- Found in src/proteus/Slices.sol: Line: 37
- Found in src/proteus/Slices.sol: Line: 40
- Found in src/proteus/Slices.sol: Line: 41
- Found in src/proteus/Slices.sol: Line: 42
- Found in src/proteus/Slices.sol: Line: 45
- Found in src/proteus/Slices.sol: Line: 46
- Found in src/proteus/Slices.sol: Line: 47
- Found in src/proteus/Slices.sol: Line: 50
- Found in src/proteus/Slices.sol: Line: 51
- Found in src/proteus/Slices.sol: Line: 52
- Found in src/proteus/Slices.sol: Line: 58
- Found in src/proteus/Slices.sol: Line: 59
- Found in src/proteus/Slices.sol: Line: 65
- Found in src/proteus/Slices.sol: Line: 66
- Found in src/proteus/Slices.sol: Line: 67
- Found in src/proteus/Slices.sol: Line: 73
- Found in src/proteus/Slices.sol: Line: 74
- Found in src/proteus/Slices.sol: Line: 75
- Found in src/proteus/Slices.sol: Line: 81
- Found in src/proteus/Slices.sol: Line: 82
- Found in src/proteus/Slices.sol: Line: 83


## NC-3: Event is missing `indexed` fields

Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

- Found in src/adapters/Curve2PoolAdapter.sol: Line: 30
- Found in src/adapters/Curve2PoolAdapter.sol: Line: 38
- Found in src/adapters/Curve2PoolAdapter.sol: Line: 46
- Found in src/adapters/CurveTricryptoAdapter.sol: Line: 35
- Found in src/adapters/CurveTricryptoAdapter.sol: Line: 43
- Found in src/adapters/CurveTricryptoAdapter.sol: Line: 51
- Found in src/ocean/Ocean.sol: Line: 111
- Found in src/ocean/Ocean.sol: Line: 141
- Found in src/ocean/Ocean.sol: Line: 142
- Found in src/ocean/Ocean.sol: Line: 143
- Found in src/ocean/Ocean.sol: Line: 151
- Found in src/ocean/Ocean.sol: Line: 159
- Found in src/ocean/Ocean.sol: Line: 160
- Found in src/ocean/OceanERC1155.sol: Line: 74
- Found in src/proteus/LiquidityPool.sol: Line: 73
- Found in src/proteus/LiquidityPool.sol: Line: 81
- Found in src/proteus/LiquidityPool.sol: Line: 89
- Found in src/proteus/LiquidityPool.sol: Line: 97
- Found in src/proteus/LiquidityPoolProxy.sol: Line: 19
- Found in src/proteus/LiquidityPoolProxy.sol: Line: 21


## NC-4: `require()` / `revert()` statements should have descriptive reason strings or custom errors



- Found in src/adapters/OceanAdapter.sol: Line: 39
- Found in src/adapters/OceanAdapter.sol: Line: 93
- Found in src/mock/MaliciousPrimitive.sol: Line: 17
- Found in src/mock/MockPrimitive.sol: Line: 19
- Found in src/mock/RecursiveMaliciousPrimitive.sol: Line: 21
- Found in src/ocean/ERC1155PermitSignatureExtension.sol: Line: 36
- Found in src/ocean/ERC1155PermitSignatureExtension.sol: Line: 47
- Found in src/ocean/Ocean.sol: Line: 198
- Found in src/proteus/EvolvingProteus.sol: Line: 203
- Found in src/proteus/EvolvingProteus.sol: Line: 307
- Found in src/proteus/EvolvingProteus.sol: Line: 313
- Found in src/proteus/EvolvingProteus.sol: Line: 340
- Found in src/proteus/EvolvingProteus.sol: Line: 346
- Found in src/proteus/EvolvingProteus.sol: Line: 375
- Found in src/proteus/EvolvingProteus.sol: Line: 384
- Found in src/proteus/EvolvingProteus.sol: Line: 410
- Found in src/proteus/EvolvingProteus.sol: Line: 417
- Found in src/proteus/EvolvingProteus.sol: Line: 444
- Found in src/proteus/EvolvingProteus.sol: Line: 451
- Found in src/proteus/EvolvingProteus.sol: Line: 478
- Found in src/proteus/EvolvingProteus.sol: Line: 485
- Found in src/proteus/EvolvingProteus.sol: Line: 582
- Found in src/proteus/EvolvingProteus.sol: Line: 584
- Found in src/proteus/EvolvingProteus.sol: Line: 644
- Found in src/proteus/EvolvingProteus.sol: Line: 647
- Found in src/proteus/EvolvingProteus.sol: Line: 818
- Found in src/proteus/LiquidityPool.sol: Line: 133
- Found in src/proteus/LiquidityPool.sol: Line: 141
- Found in src/proteus/LiquidityPool.sol: Line: 396
- Found in src/proteus/LiquidityPool.sol: Line: 397
- Found in src/proteus/LiquidityPool.sol: Line: 398
- Found in src/proteus/LiquidityPool.sol: Line: 419
- Found in src/proteus/LiquidityPool.sol: Line: 429
- Found in src/proteus/Proteus.sol: Line: 70
- Found in src/proteus/Proteus.sol: Line: 73
- Found in src/proteus/Proteus.sol: Line: 94
- Found in src/proteus/Proteus.sol: Line: 97
- Found in src/proteus/Proteus.sol: Line: 120
- Found in src/proteus/Proteus.sol: Line: 124
- Found in src/proteus/Proteus.sol: Line: 146
- Found in src/proteus/Proteus.sol: Line: 150
- Found in src/proteus/Proteus.sol: Line: 173
- Found in src/proteus/Proteus.sol: Line: 177
- Found in src/proteus/Proteus.sol: Line: 200
- Found in src/proteus/Proteus.sol: Line: 204
- Found in src/proteus/Proteus.sol: Line: 319
- Found in src/proteus/Proteus.sol: Line: 388
- Found in src/proteus/Proteus.sol: Line: 392
- Found in src/proteus/Proteus.sol: Line: 720
- Found in src/proteus/Slices.sol: Line: 30
- Found in src/proteus/Slices.sol: Line: 31
- Found in src/proteus/Slices.sol: Line: 32
- Found in src/proteus/Slices.sol: Line: 33




### Time spent:
1 hours