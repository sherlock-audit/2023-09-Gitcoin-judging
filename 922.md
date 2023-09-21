Scruffy Taupe Orca

medium

# Allowed tokens checks do not work correctly in `DonationVotingMerkleDistributionBaseStrategy`
## Summary
`DonationVotingMerkleDistributionBaseStrategy` accounts for allowed tokens in an array. It is used to prevent allocating using disallowed tokens.

## Vulnerability Detail 
There is a problem with the checks for those tokens. 

If you try to fund the pool directly from the Allo contract via `_fundPool()` function there aren't any checks if the token is allowed or not. The amount is being sent directly to the strategy contract.

Also the checks in the `_allocate` function in `DonationVotingMerkleDistributionBaseStrategy` aren't working properly.

### Proof of concept

When initializing the strategy allowedTokens array is being set:

```solidity
288:        uint256 allowedTokensLength = _initializeData.allowedTokens.length;
289:
290:        if (allowedTokensLength == 0) {
291:            // all tokens
292:            allowedTokens[address(0)] = true;
293:        }
294:
295:        for (uint256 i; i < allowedTokensLength; i++) {
296:            allowedTokens[_initializeData.allowedTokens[i]] = true;
297:        }
```
https://github.com/sherlock-audit/2023-09-Gitcoin/blob/6430c8004017e96ae2f5aac365bdefd0b6eeea72/allo-v2/contracts/strategies/donation-voting-merkle-base/DonationVotingMerkleDistributionBaseStrategy.sol#L288-L302

and on `_allocate` the token provided is being validated using this if check:

```solidity
653        if (!allowedTokens[token] && !allowedTokens[address(0)]) {
654            revert INVALID();
655        }
```
https://github.com/sherlock-audit/2023-09-Gitcoin/blob/6430c8004017e96ae2f5aac365bdefd0b6eeea72/allo-v2/contracts/strategies/donation-voting-merkle-base/DonationVotingMerkleDistributionBaseStrategy.sol#L653-L655

Let's say the allowedTokens array is empty and we execute `_allocate` where token provided is `USDT`

The validation in `_allocate` would look like:
`if (!false && !true)` -> won't revert and it would continue the execution of the function like normal.

## Impact
Allowed tokens restrictions won't function correctly.

## Code Snippet
https://github.com/sherlock-audit/2023-09-Gitcoin/blob/6430c8004017e96ae2f5aac365bdefd0b6eeea72/allo-v2/contracts/core/Allo.sol#L502-L520
https://github.com/sherlock-audit/2023-09-Gitcoin/blob/6430c8004017e96ae2f5aac365bdefd0b6eeea72/allo-v2/contracts/strategies/donation-voting-merkle-base/DonationVotingMerkleDistributionBaseStrategy.sol#L288-L302
https://github.com/sherlock-audit/2023-09-Gitcoin/blob/6430c8004017e96ae2f5aac365bdefd0b6eeea72/allo-v2/contracts/strategies/donation-voting-merkle-base/DonationVotingMerkleDistributionBaseStrategy.sol#L653-L655

## Tool used
Manual review

## Recommendation
Add some sort of validation of allowed tokens in `_fundPool()` and for `_allocate()` it is better to use `||` instead of `&&` on the if statement.