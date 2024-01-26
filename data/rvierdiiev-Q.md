## QA-01 `setRewards` allows to set rewards for epochs in the past
### Description
`LendingLedger.setRewards` accepts `_fromEpoch` and `_toEpoch` params. The only check that function does is if epoch [are set as weeks](https://github.com/code-423n4/2024-01-canto/blob/main/src/LendingLedger.sol#L128). But there is no check if rewards are set for previous epochs.
### Recommendation
Check that provided epochs are in future

## QA-02 If `setRewards` will be called for same epochs, then `cantoPerBlock` will be rewritten
### Description
`LendingLedger.setRewards` accepts `_fromEpoch` and `_toEpoch` params. The only check that function does is if epoch [are set as weeks](https://github.com/code-423n4/2024-01-canto/blob/main/src/LendingLedger.sol#L128). It allows to provide epochs that were already set before. In case if this will happen, which is likely not the case as sponsor claimed, then [`cantoPerBlock` will be rewritten](https://github.com/code-423n4/2024-01-canto/blob/main/src/LendingLedger.sol#L130) for the epoch.
### Recommendation
Increase `cantoPerBlock` for the epoch.
`cantoPerBlock[i] += _amountPerBlock;`