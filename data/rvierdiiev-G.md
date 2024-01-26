## G-01 Don't make variables public unless necessary
### Description
All varibles inside `LendingLedger` are public. This comes with some overhead costs that can be avoided for some variables. A public variable creates a public getter function of the same name and increases contract size and deployment cost.

I think that `BLOCK_EPOCH` and `lendingMarketTotalBalance` variables can be private as they will likely not be used.

### G-02 Use while loop instead of for loop
## Description
`LendingLedger.setRewards` [uses for loop](https://github.com/code-423n4/2024-01-canto/blob/main/src/LendingLedger.sol#L129-L131) to set block rewards for the epoch. While loops consume less gas than for loop, so implementetion can be changed  to save gas.

### G-03 Move variables declaration outside the loop
## Description
`update_market` function loops through passed epochs and inside the loop each cycle it declares [some variables](https://github.com/code-423n4/2024-01-canto/blob/main/src/LendingLedger.sol#L64-L67). You can save some gas in case if you move those variables declaration out of loop, so they are not created each time.

### G-04 Use smaller error symbols
## Description
Error message that is more than 32 bytes costs more gas. There are several places in the code, where `require` is used for checking. They revert with long custom message. You can revert with some code instead.
```solidity
require(lendingMarketWhitelist[_market], "Market not whitelisted");

require(updatedMarketBalance >= 0, "Market balance underflow");

require(success, "Failed to send CANTO");

require(_fromEpoch % BLOCK_EPOCH == 0 && _toEpoch % BLOCK_EPOCH == 0, "Invalid block number");
```

Also you can use assembly to make revert more cheaper.