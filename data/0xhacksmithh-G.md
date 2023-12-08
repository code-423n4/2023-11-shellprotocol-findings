### [Gas-0] Unnecessary Calculation

```diff
```
```
```

### [Gas-0] Could be initialized with `_ERC1155InteractionStatus` & `_ERC721InteractionStatus` Non-zero values

```diff
```
```
```

### [Gas-0] Event Could emited with more gas efficient 

```diff
```
```
```

### [Gas-0] Instead of caching `_getSpecifiedToken()`, this could directly used inside follow up function `_executeInteraction()`

```diff
```
```
```

### [Gas-0] Initialize with non-zero number

```diff
```
```
```

### [Gas-0] `_idLength = ids.length;` should written one step above so that gas for extra `ids.length` calculation could saved

```diff
```
```
```

### [Gas-0] Caching `ids.length` costs more gas than using as it is loops

```diff
```
```
```

### [Gas-0] No need to Cache `userAddress` to storage

```diff
```
```
```

### [Gas-0] Create Variable out side of loop and override it with each iteration

```diff
```
```
```

### [Gas-0] `_calculateUnwrapFee()` could directly used for follow up calculation with-out caching it

```diff
```
```
```

### [Gas-0] Storage could be packed to save extra slot

```diff
```
```
```


### [Gas-0] No Need To cache mathematical operation i.e `uint256 amountRemaining = amount - feeCharged`

```diff
```
```
```

### [Gas-0] Use of `&&` and `||` operators in `if` condition

```diff
```
```
```

### [Gas-0] `+= 1` Could replace with `unchecked {++}`

```diff
```
```
```

### [Gas-0] Some Substractions `-` could marked as uncheck due to previous condition check.

```diff
```
```
```

### [Gas-0] No Need to assign `0` to `dust` & `truncatedAmount` in functions calculation as it's default value already `0` 

```diff
```
```
```

### [Gas-0] Uncheck `%` operation 

```diff
```
```
```

### [Gas-0] 

```diff
```
```
```

### [Gas-0] 

```diff
```
```
```

### [Gas-0] 

```diff
```
```
```

### [Gas-0] 

```diff
```
```
```