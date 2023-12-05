If trying to invoke `changeUnwrapFee` with `nextUnwrapFeeDivisor` equal to `MIN_UNWRAP_FEE_DIVISOR` transaction should revert because it is same number we will not change anything.

```diff
-if (MIN_UNWRAP_FEE_DIVISOR > nextUnwrapFeeDivisor) revert();
+if (MIN_UNWRAP_FEE_DIVISOR >= nextUnwrapFeeDivisor) revert();
```