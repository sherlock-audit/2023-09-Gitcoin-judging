Fierce Pearl Falcon

medium

# Funding During Distribution Can Skew Allocation Amounts

Changes to the funding pool during the fund distribution process can result in an inconsistent and incorrect allocation of funds to recipients.

## Vulnerability Detail

In the Allo contract, the `fundPool` function allows for the addition of funds to a pool. This function can be invoked by anyone at any time, thereby increasing the `poolAmount`.

                function fundPool(uint256 _poolId, uint256 _amount) external payable nonReentrant {
                        // if amount is 0, revert with 'NOT_ENOUGH_FUNDS()' error
                        if (_amount == 0) revert NOT_ENOUGH_FUNDS();

                        // Call the internal fundPool() function
                        _fundPool(_amount, _poolId, pools[_poolId].strategy);
                }

https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/core/Allo.sol#L339-L345

However, in the QVSimpleStrategy, fund distribution to recipients is based on the `poolAmount` and the proportion of votes that each recipient has received.

                if (!paidOut[_recipientId] && totalRecipientVotes != 0) {
                        amount = poolAmount * recipient.totalVotesReceived / totalRecipientVotes;
                }

https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/qv-base/QVBaseStrategy.sol#L570-L572

The issue arises when not all recipients are paid out in a single transaction due to a high number of recipients. If additional funds are added to the pool between these transactions, it results in a skewed distribution. Recipients who are paid later may end up receiving disproportionately more funds than those paid earlier, despite having fewer votes.

## Impact

This discrepancy results in an unfair and incorrect distribution of funds among the recipients.

## Code Snippet

https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/core/Allo.sol#L339-L345
https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/qv-base/QVBaseStrategy.sol#L570-L572

## Tool used

Manual Review

## Recommendation

Temporarily disable the `fundPool` function during the fund distribution process.