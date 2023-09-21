Furry Cider Panda

medium

# If an allocator is removed, the votes already cast by that allocator should also be deleted
## Summary

The Pool manager can remove an allocator via [[QVSimpleStrategy.removeAllocator](https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/qv-simple/QVSimpleStrategy.sol#L97-L101)](https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/qv-simple/QVSimpleStrategy.sol#L97-L101). If the removed allocator hasn't voted yet, that's no problem. However, if the allocator has already voted for one or more recipients, these votes should be deleted. Because the votes cast by the removed allocator no longer have any meaning. The payout each recipient can get depends on the number of votes they receive. `The more votes recipient gets, the more payout recipient gets`. In this way, the distribution of funds can be more equitable.

## Vulnerability Detail

Consider the following scenario:

For simplicity, there are three allocators (alice, bob, tony), `maxVoiceCreditsPerAllocator` is 100. There are two recipients (A and B).

1.  alice votes for A with 100 VoiceCredits by calling `Allo.allocate`. After that, A got 10 votes (`sqrt(100) = 10`).
2.  bob votes for B with 100 VoiceCredits by calling `Allo.allocate`. After that, B got 10 votes (`sqrt(100) = 10`).
3.  tony votes for B with 100 VoiceCredits by calling `Allo.allocate`. After that, B got another 10 votes (`sqrt(100) = 10`). So, B got 20 votes totally.
4.  The total number of votes is `20 + 10 = 30`.
5.  The pool manager calls removeAllocator to remove tony.
6.  The pool manager calls `Allo.distribute` to distribute tokens for A and B. The amount of distributed tokens is calculated by [[_getPayout](https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/qv-base/QVBaseStrategy.sol#L571C22-L571C32)](https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/qv-base/QVBaseStrategy.sol#L571C22-L571C32): `payoutA = poolAmount * (10/30) = poolAmount * 0.33`, `payoutB = poolAmount * (20/30) = poolAmount * 0.66`.

We can see that B got 66% of the funds, while A only got 33% of the funds. **Such distribution is unfair. Because the votes already cast by the removed allocator (tony) should be deleted. In other words, A and B should share the funds equally**.

## Impact

The votes cast by the removed allocator still affects the distribution of funds. These votes should have been deleted, but instead they were counted.

## Code Snippet

https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/qv-simple/QVSimpleStrategy.sol#L97-L101

## Tool used

Manual Review

## Recommendation

Because at the end of [[QVBaseStrategy._qv_allocate](https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/qv-base/QVBaseStrategy.sol#L533)](https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/qv-base/QVBaseStrategy.sol#L533), the `Allocated(_recipientId, voteResult, _sender)` event will be emitted, and `_sender` here is allocator. Therefore, an off-chain program can collect every vote information of every allocator. Then we can add parameters for `removeAllocator` to delete the votes that have been cast.