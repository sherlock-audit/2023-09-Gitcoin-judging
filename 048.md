Rapid Lead Cricket

medium

# Exponential Inflation of Voice Credits in Quadratic Voting Strategy
## Summary
The current implementation of the `_qv_allocate` function in the quadratic voting strategy lead to an exponential inflation of voice credits cast to a recipient due to repeated addition of previous allocations. This will significantly distort the voting outcome and undermine the integrity of the voting system

## Vulnerability Detail
In the given code snippet, we observe a potential issue in the way voice credits are being accumulated for each recipient. The specific lines of code in question are:
```solidity
function _qv_allocate(
        ...
    ) internal onlyActiveAllocation {
        ...
        uint256 creditsCastToRecipient = _allocator.voiceCreditsCastToRecipient[_recipientId];
        ...
        // get the total credits and calculate the vote result
        uint256 totalCredits = _voiceCreditsToAllocate + creditsCastToRecipient;
        ...
        //E update allocator mapping voice for this recipient
        _allocator.voiceCreditsCastToRecipient[_recipientId] += totalCredits; //E @question should be only _voiceCreditsToAllocate
        ...
    }
```
We can see that at the end : 
```solidity
_allocator.voiceCreditsCastToRecipient[_recipientId] = _allocator.voiceCreditsCastToRecipient[_recipientId] + _voiceCreditsToAllocate +  _allocator.voiceCreditsCastToRecipient[_recipientId];
```

Here, totalCredits accumulates both the newly allocated voice credits (`_voiceCreditsToAllocate`) and the credits previously cast to this recipient (`creditsCastToRecipient`). Later on, this totalCredits is added again to `voiceCreditsCastToRecipient[_recipientId]`, thereby including the previously cast credits once more

### Proof of Concept (POC):
Let's consider a scenario where a user allocates credits in three separate transactions:

1. Transaction 1: Allocates 5 credits
- creditsCastToRecipient initially is 0
- totalCredits = 5 (5 + 0)
- New voiceCreditsCastToRecipient[_recipientId] = 5

2. Transaction 2: Allocates another 5 credits
- creditsCastToRecipient now is 5 (from previous transaction)
- totalCredits = 10 (5 + 5)
- New voiceCreditsCastToRecipient[_recipientId] = 15 (10 + 5)

3. Transaction 3: Allocates another 5 credits
- creditsCastToRecipient now is 15
- totalCredits = 20 (5 + 15)
- New voiceCreditsCastToRecipient[_recipientId] = 35 (20 + 15)

From the above, we can see that the voice credits cast to the recipient are exponentially growing with each transaction instead of linearly increasing by 5 each time

## Impact
Exponential increase in the voice credits attributed to a recipient, significantly skewing the results of the voting strategy( if one recipient receive 15 votes in one vote and another one receive 5 votes 3 times, the second one will have 20 votes and the first one 15)
Over time, this could allow for manipulation and loss of trust in the voting mechanism and the percentage of amount received by recipients as long as allocations are used to calculate the match amount they will receive from the pool amount.

## Code Snippet

https://github.com/allo-protocol/allo-v2/blob/main/contracts/strategies/qv-base/QVBaseStrategy.sol#L529

## Tool used

Manual Review

## Recommendation
Code should be modified to only add the new voice credits to the recipient's tally. The modified line of code should look like:
```solidity
_allocator.voiceCreditsCastToRecipient[_recipientId] += _voiceCreditsToAllocate;
```