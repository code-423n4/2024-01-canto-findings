## Findings Summary

| Label | Description |
| - | - |
| [L-01](#) |Unable to retrieve remaining CANTO after LendingLedger is deprecated |
| [L-02](#) |claim() needs to be able to specify the recipient to avoid some users being unable to receive CANTO|
| [L-03](#) |Increase reward debt should use round up |

## Unable to retrieve remaining CANTO after LendingLedger is deprecated
When `LendingLedger` is in use, `CANTO` needs to be transferred into `LendingLedger.sol` so that users can execute `claim()` after rewards accumulate. 
If `LendingLedger` is deprecated, the `CANTO` that has been transferred in cannot be transferred out. 
Also, `CANTO` that is accidentally transferred in cannot be transferred out 
because `LendingLedger` does not provide any method to transfer out `CANTO`. 
It is suggested to add a method for the administrator to transfer out.
```diff
+    function recover() external onlyGovernance{
+        (bool success, ) = msg.sender.call{value: address(this).balance}("");
+       require(success, "Failed to send CANTO");
+    }    
```

## [L-02] claim() needs to be able to specify the recipient to avoid some users being unable to receive CANTO
When the current user executes `claim()`, only `msg.sender` can receive it, and the recipient cannot be specified.
```solidity
    function claim(address _market) external {
...
        if (cantoToSend > 0) {
            (bool success, ) = msg.sender.call{value: uint256(cantoToSend)}("");
            require(success, "Failed to send CANTO");
        }
    }
```

If it is a contract user, they may not have `receive()` and cannot receive. It is suggested to add the ability to specify the recipient.
```diff
-   function claim(address _market) external {
+   function claim(address _market , address _to) external {
...
        if (cantoToSend > 0) {
-           (bool success, ) = msg.sender.call{value: uint256(cantoToSend)}("");
+           (bool success, ) = _to.call{value: uint256(cantoToSend)}("");
            require(success, "Failed to send CANTO");
        }
    }
```

## [L-03] Increase reward debt should use round up
Currently, `round down` is used when increasing `reward debt`.
```solidity
    function sync_ledger(address _lender, int256 _delta) external {
        address lendingMarket = msg.sender;
        update_market(lendingMarket); // Checks if the market is whitelisted
        MarketInfo storage market = marketInfo[lendingMarket];
        UserInfo storage user = userInfo[lendingMarket][_lender];

        if (_delta >= 0) {
            user.amount += uint256(_delta);
@>          user.rewardDebt += int256((uint256(_delta) * market.accCantoPerShare) / 1e18);
            user.secRewardDebt += int256((uint256(_delta) * market.secRewardsPerShare) / 1e18);
        } else {
...
```
But when the user performs `claim()`, it may not `round down`.
```solidity
    function claim(address _market) external {
...
@>      int256 accumulatedCanto = int256((uint256(user.amount) * market.accCantoPerShare) / 1e18);
@>      int256 cantoToSend = accumulatedCanto - user.rewardDebt;
```
This may result in the user getting a few more `wei` in `rewards` than expected. 
It is suggested to modify `reward debt` to use round up.
```diff
    function sync_ledger(address _lender, int256 _delta) external {
        address lendingMarket = msg.sender;
        update_market(lendingMarket); // Checks if the market is whitelisted
        MarketInfo storage market = marketInfo[lendingMarket];
        UserInfo storage user = userInfo[lendingMarket][_lender];

        if (_delta >= 0) {
            user.amount += uint256(_delta);
-           user.rewardDebt += int256((uint256(_delta) * market.accCantoPerShare) / 1e18);
+           user.rewardDebt += int256(Math.ceilDiv(uint256(_delta) * market.accCantoPerShare) , 1e18));
            user.secRewardDebt += int256((uint256(_delta) * market.secRewardsPerShare) / 1e18);
        } else {
...
```