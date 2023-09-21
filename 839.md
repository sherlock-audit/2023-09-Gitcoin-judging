Rhythmic Rusty Swan

medium

# When ```block.timestamp == allocationEndTime``` this can bypassed both ```onlyActiveAllocation()``` and ```onlyAfterAllocation()``` modifiers.
## Summary
On the condition that ```block.timestamp == allocationEndTime``` both modifiers can be bypassed as neither of them revert on this condition. This can lead to unwanted behaviour as it is unclear whether the protocol wants this specific time to be part of the allocation process or not.

## Vulnerability Detail
The distribute based functions should only be called after all the allocation has taken place and the allocate functions should only be called during active allocation times, however this specific condition at  ```block.timestamp == allocationEndTime``` has left it so that both functions can be called at the same time. 

## Impact
This could lead to a distribution function being called before an allocation has taken place which seems to be unwanted behaviour for this protocol.

## Code Snippet

https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/donation-voting-merkle-base/DonationVotingMerkleDistributionBaseStrategy.sol#L465-L469

https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/donation-voting-merkle-base/DonationVotingMerkleDistributionBaseStrategy.sol#L473-L476

These modifiers are also used in QVBaseStrategy.sol.

## Tool used

Manual Review

## Recommendation

I would recommend that the protocol make it so that the ```_checkOnlyAfterAllocation()``` function reverts at the time ```block.timestamp == allocationEndTime```. 

Change the if statement from ```if (block.timestamp < allocationEndTime)``` to:

```if (block.timestamp <= allocationEndTime)``` so that it is strictly only after the allocation period has ended.

