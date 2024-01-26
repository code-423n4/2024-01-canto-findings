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

## QA-03 Contract doesn't emit any event
### Description
`LendingLedger` doesn't emit any event which can help tracking activity for offline services. For example when market is whitelisted/blacklisted, when rewards per block is set for new epochs, when user claimed rewards or when governance has changed.
### Recommendation
Add events.

## QA-04 Loss of rewards for users in case if market is blacklisted
### Description
In case if market was whitelisted, earned rewards and then [became blacklisted](https://github.com/code-423n4/2024-01-canto/blob/main/src/LendingLedger.sol#L137-L143), then it will be not possible for users to claim already earned rewards anymore, because `update_market` function revert for non whitelisted markets.
### Recommendation
Give ability for users to claim already accrued rewards for blacklisted market. Maybe you should remove `require` in the `update_market` function. As i guess that blacklisted market will have 0 weight in gauge.

## QA-05 Some rewards will be lost in case if market will be blacklisted
### Description
Any market [can be blacklisted](https://github.com/code-423n4/2024-01-canto/blob/main/src/LendingLedger.sol#L137-L143). This means that rewards [will not accrue for it anymore](https://github.com/code-423n4/2024-01-canto/blob/main/src/LendingLedger.sol#L57). 

When market is blacklisted, then `update_market` is not called for it for last time(and already accrued rewards are not available for users as i described in QA-04). And those undistributed earned rewards are not tracked in any way to be distributed to other markets. Thus, those funds will just stuck.
### Recommendation
You need to allow user to claim all funds(and when blacklisting you should call `update_market` for last time) or you need to track unclaimed funds and redistribute them other markets.