Future Sangria Giraffe

high

# Missing access modifier for `RFPSimpleStrategy.setPoolActive()` may lead to multiple issues

`RFPSimpleStrategy.setPoolActive()` can be called by anybody since it's missing the `onlyPoolManager(msg.sender)` modifier, which can be abused by a malicious actor to steal funds.

## Vulnerability Detail

The comment on line 217 in RFPSimpleStrategy.sol says that `'msg.sender' must be a pool manager` in order to be able to call `RFPSimpleStrategy.setPoolActive()`. However, the necessary `onlyPoolManager(msg.sender)` modifier is missing.

## Impact

Multiple functions inside `RFPSimpleStrategy.sol` are either using the `onlyActivePool` or the `onlyInactivePool` modifiers:

* `RFPSimpleStrategy._distribute()`
* `RFPSimpleStrategy.withdraw()`
* `RFPSimpleStrategy._registerRecipient()`
* `RFPSimpleStrategy._allocate()`

A malicious actor (Alice) might do the following for example:

1. Alice registers themself as recipient for a `RFPSimpleStrategy`, specifying a `proposalBid` which is `15e18`.
1. Alice is being declared as the accepted recipient by the pool manager.
1. Now if the tokens were distributed to Alice, the amount of tokens Alice would receive would be `(15e18 * milestone.amountPercentage) / 1e18` (line 435 RFPSimpleStrategy.sol).
1. However, Alice calls `RFPSimpleStrategy.setPoolActive()` to make the pool active again, before the tokens are distributed. Alice might do this by either frontrunning or by executing the tx earlier.
1. Now Alice can call `RFPSimpleStrategy._registerRecipient()`, since the pool is active again, and Alice re-registers themself but with a higher `proposalBid` than was accepted before (line 378 RFPSimpleStrategy.sol), for example they re-register with a `proposalBid` of `60e18`.
1. Then Alice calls `RFPSimpleStrategy.setPoolActive()` to set the pool inactive, so that the tokens can be distributed.
1. Now when the tokens are distributed to Alice for the first milestone (and later also for subsequent milestones), they receive a much higher amount of tokens, since Alice maliciously increased their accepted `proposalBid` from `15e18` to `60e18`, so they would now receive `(60e18 * milestone.amountPercentage) / 1e18` (line 435 RFPSimpleStrategy.sol) which is more than was accepted.

The above example illustrates how Alice can abuse setting the pool to active and inactive to change their accepted `proposalBid` to receive more tokens.

Also, Alice could potentially steal funds from the strategy, if they get accepted with a smaller `proposalBid` and then maliciously increase the `proposalBid` as described in the above example, so that Alice would receive a much higher amount of tokens that they are not eligible to receive and that are effectively being stolen from the funds of the strategy.

## Code Snippet

https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/rfp-simple/RFPSimpleStrategy.sol#L217-L221

https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/rfp-simple/RFPSimpleStrategy.sol#L417-L450

https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/rfp-simple/RFPSimpleStrategy.sol#L314-L380

https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/rfp-simple/RFPSimpleStrategy.sol#L386-L393

https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/rfp-simple/RFPSimpleStrategy.sol#L295


## Tool used

Manual Review

## Recommendation

Consider adding the missing access modifier `onlyPoolManager(msg.sender)` to `RFPSimpleStrategy.setPoolActive()`.