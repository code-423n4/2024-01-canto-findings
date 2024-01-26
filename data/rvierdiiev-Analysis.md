## Overview 
Canto protocol has redesigned LendingLedger contract to allow users to claim their rewards at any time, so they should not wait when epoch will finish. Also `secRewardDebt` variable was introduced to allow third parties to distribute rewards to users based on changes of that value during the time.

## Key Components

`LendingLedger` contract allows governance to set rewards for the epochs, which should be distributed among whitelisted markets according to their weight in gauge controller. `LendingLedger` contract tracks total supply of each whitelisted market in order to calculate correct reward per share for the users. Market's participants can receive rewards according to the amount of shares that they have in the market. Also each user has `secRewardDebt` variable that is calculated to allow third parties to rewards users.

## Systemic Risks

`Correct decisions from governance`: whitelisted markets and rewards per epoch are set by governance. Thus governance is responsible to validate provided values as it possible to break intended work with incorrect input.

## Centralization risk

`LendingLedger` has only governance role, which should belong to the DAO. No other party is allowed to change contract settings.

### Time spent:
4 hours

### Time spent:
4 hours