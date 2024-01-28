## Low

### [L-01] Hard-coded block rate

If the block generation rate ever changes in the Canto network, it will lead to epochs being shorter or longer than expected, which affects directly to the amount of CANTO rewards and secundary rewards per share:

[**LendingLedger, line 11**](https://github.com/code-423n4/2024-01-canto/blob/5e0d6f1f981993f83d0db862bcf1b2a49bb6ff50/src/LendingLedger.sol#L11)

```solidity
    uint256 public constant BLOCK_EPOCH = 100_000; // 100000 blocks, roughly 1 week
```

[**LendingLedger, lines 64 to 71**](https://github.com/code-423n4/2024-01-canto/blob/5e0d6f1f981993f83d0db862bcf1b2a49bb6ff50/src/LendingLedger.sol#L64C1-L71C111)

```solidity
    function update_market(address _market) public {
        ...

                    uint256 epoch = (i / BLOCK_EPOCH) * BLOCK_EPOCH; // Rewards and voting weights are aligned on a weekly basis
                    uint256 nextEpoch = i + BLOCK_EPOCH;
                    uint256 blockDelta = Math.min(nextEpoch, block.number) - i;
                    uint256 cantoReward = (blockDelta *
                        cantoPerBlock[epoch] *
                        gaugeController.gauge_relative_weight_write(_market, epoch)) / 1e18;
                    market.accCantoPerShare += uint128((cantoReward * 1e18) / marketSupply);
                    market.secRewardsPerShare += uint128((blockDelta * 1e18) / marketSupply); // TODO: Scaling
        
        ...
        
    }
```

Consider making it a non-constant variable which can be set by governance:

```solidity
    uint256 public BLOCK_EPOCH = 100_000; // 100000 blocks, roughly 1 week

    function setBlockEpoch(uint256 epoch) external onlyGovernance {
        BLOCK_EPOCH = epoch;
    }
```

## Non-critical

### [NC-01] No error string in `require`

[**LendingLedger, modifier onlyGovernance**](https://github.com/code-423n4/2024-01-canto/blob/5e0d6f1f981993f83d0db862bcf1b2a49bb6ff50/src/LendingLedger.sol#L41)

```solididty
    modifier onlyGovernance() {
        require(msg.sender == governance);
        _;
    }
```

Change it to something like:

```solididty
    modifier onlyGovernance() {
        require(msg.sender == governance, "Not authorized");
        _;
    }
```

### [NC-02] Missing documentation in `update_market`

There are no comments explaining what does [update_marked](https://github.com/code-423n4/2024-01-canto/blob/5e0d6f1f981993f83d0db862bcf1b2a49bb6ff50/src/LendingLedger.sol#L56). Consider adding a brief explanation.

### [NC-03] Missing events emitted in critical areas

Consider emitting events when doing changes to state variables like `governance` or when a market is being whitelisted, so that off-chain actors can keep track of the contract state:

### [NC-04] Unused imports

Namely, `VotingEscrow`, `IERC20` and `SafeERC20`:

```solidity
import {VotingEscrow} from "./VotingEscrow.sol";
import {GaugeController} from "./GaugeController.sol";
import {Math} from "@openzeppelin/contracts/utils/math/Math.sol";
import {IERC20, SafeERC20} from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
```
