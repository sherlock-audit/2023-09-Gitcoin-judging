Silly Carob Opossum

medium

# RFPSimpleStrategy milestones can be set multiple times
## Summary

Until the first distribution is completed, it's possible to call `setMilestones` function multiple times. New milestones are added to the previous ones. The `totalAmountPercentage` of all milestones in this case will be greater than 100%. It also affects all the contracts that are inherited from RFPSimpleStrategy.

## Vulnerability Detail

The `setMilestones` function in `RFPSimpleStrategy` contract checks if `MILESTONES_ALREADY_SET` or not by `upcomingMilestone` index.

```solidity
if (upcomingMilestone != 0) revert MILESTONES_ALREADY_SET();
```

But `upcomingMilestone` increases only after distribution, and until this time will always be equal to 0.

## Impact

It can accidentally break the pool state or be used with malicious intentions.

1. Two managers accidentally set the same milestones. Milestones are duplicated and can't be reset, the pool needs to be recreated.
2. The manager, in cahoots with the recipient, sets milestones one by one, thereby bypassing `totalAmountPercentage` check and increasing the payout amount.

## Code Snippet

https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/rfp-simple/RFPSimpleStrategy.sol#L224-L247

## Tool used

Manual Review

## Recommendation

Fix condition if milestones should only be set once. 

```solidity
if (milestones.length > 0) revert MILESTONES_ALREADY_SET();
```

Or allow milestones to be reset while they are not in use.


```solidity
if (milestones.length > 0) {
    if (milestones[0].milestoneStatus != Status.None) revert MILESTONES_ALREADY_IN_USE();
    delete milestones;
}
```
