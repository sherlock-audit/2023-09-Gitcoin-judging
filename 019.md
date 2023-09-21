Fancy Khaki Perch

medium

# `fundPool` does not work with fee-on-transfer token
## Summary
`fundPool` does not work with fee-on-transfer token
## Vulnerability Detail
In `_fundPool`, the parameter for `increasePoolAmount` is directly the amount used in the `transferFrom` call.

https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/core/Allo.sol#L516-L517
```solidity
        _transferAmountFrom(_token, TransferData({from: msg.sender, to: address(_strategy), amount: amountAfterFee}));
        _strategy.increasePoolAmount(amountAfterFee);
```

When `_token` is a fee-on-transfer token, the actual amount transferred to `_strategy` will be less than `amountAfterFee`. Therefore, the current approach could lead to a recorded balance that is greater than the actual balance.
## Impact
`fundPool` does not work with fee-on-transfer token
## Code Snippet
https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/core/Allo.sol#L516-L517
## Tool used

Manual Review

## Recommendation
Use the change in `_token` balance as the parameter for `increasePoolAmount`.