Ambitious Brick Ladybug

medium

# Inaccurate poolAmount calculations with inflationary, deflationary and rebasing tokens
## Summary
The protocol allows for the usage of any ERC-20 token, including those with inflationary, deflationary, or rebasing properties. The system's logic assumes a static value for the pool's balance (`poolAmount`) across various functions, which can result in discrepancies if the underlying token's balance changes unexpectedly.
```solidity 
    function increasePoolAmount(uint256 _amount) external override onlyAllo {
        _beforeIncreasePoolAmount(_amount);
        poolAmount += _amount;
        _afterIncreasePoolAmount(_amount);
    }

```
## Vulnerability Detail
When using inflationary, deflationary, or rebasing tokens:

* Inflationary Tokens: These tokens increase in supply over time. If an inflationary token is used as the pool's token, the actual balance of the token can become greater than the stored poolAmount.

* Deflationary Tokens: These tokens reduce in supply, usually by burning a percentage on each transfer. When distributing funds or calculating fees with deflationary tokens, the actual amount received by recipients or the treasury might be less than expected due to the burn mechanism. The poolAmount might then be greater than the actual token balance.

* Rebasing Tokens: The balance of rebasing tokens can change based on external factors, such as the price of the token relative to another asset. This can lead to a change in the token balance without any transfer or explicit action taken by the contract.

In the protocol:

* The `_fundPool` function from Allo contract transfers funds based on a given amount, increasing the `poolAmount` variable, which doesn't account for any potential changes in the token's balance due to its inflationary, deflationary, or rebasing nature.
* The `_distributeSingle` in DonationVotingMerkleDistributionBaseStrategy.sol, `_distribute` in QVBaseStrategy.sol, and `_distribute` in RFPSimpleStrategy.sol functions all use `poolAmount` to calculate distributions and check for sufficient funds. 
* Additionally, the `withdraw` function from DonationVotingMerkleDistributionBaseStrategy contract reverts if the desired amount is greater that the `poolAmount` value, preventing the token from being withdrawn.

## Impact
If the balance of tokens increases, the protocol might incorrectly think there are insufficient funds, preventing valid distributions of the whole amount of funds.
If the balance of tokens decreases, the protocol might mistakenly believe there are enough funds, leading to transaction reverts.
## Code Snippet
https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/core/Allo.sol#L502-L520
https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/donation-voting-merkle-base/DonationVotingMerkleDistributionBaseStrategy.sol#L790
https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/rfp-simple/RFPSimpleStrategy.sol#L438
https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/qv-base/QVBaseStrategy.sol#L571
## Tool used

Manual Review

## Recommendation
Implement a function in the Allo or BaseStrategy contract, that allows an admin or the pool owner to manually synchronize the `poolAmount` with the actual token balance.