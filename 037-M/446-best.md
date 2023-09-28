Future Sangria Giraffe

medium

# `QVSimpleStrategy`: If someone fund a pool when the fund is partially/fully distributed, part of the fund may be locked
## Summary

When `Allo::fundPool` is called when the funds are partially or fully distributed, the added funds may be locked.

## Vulnerability Detail

`Allo::fundPool` can be called by anyone at anytime, and it will increase the `BaseStrategy.poolAmount` via `BaseStrategy::increasePoolAmount()`.

https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/core/Allo.sol#L339-L345

The `BaseStrategy.poolAmount` storage variable is used to determine the payout of each recipient by `QVBaseStrategy::_getPayout`:

https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/qv-base/QVBaseStrategy.sol#L559-L574


When the fund is distributed by the pool manager via `QVBaseStrategy::_distribute`, the `paidOut` flag for the recipientId whose share was distributed will be set to be true.

https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/qv-base/QVBaseStrategy.sol#L458

The problem occurs when some funds are  added when some funds are distributed.
In the case, the funds will be partially or fully locked.

## Impact

If some funds are added after the distribution is started, the added funds may be locked.

## Code Snippet

https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/core/Allo.sol#L339-L345

https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/qv-base/QVBaseStrategy.sol#L559-L574


https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/qv-base/QVBaseStrategy.sol#L458

## Tool used

Manual Review

## Recommendation

Consider adding recoverFund function like other strategies.
Alternatively, allow the `fundPool` function only before the distribution starts.

