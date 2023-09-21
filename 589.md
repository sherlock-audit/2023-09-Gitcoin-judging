Dandy Lavender Wombat

medium

# In `QVBaseStrategy` the same manager can review one applicant multiple times and thereby render the safety measure of `reviewThreshold` useless
## Summary
In `QVBaseStrategy` potential recipients are reviewed by poolManager. For a recipient to be accepted, the number of managers that review him with a particular status needs to cross the `reviewThreshold`. The problem is that one manager can review a participant multiple times and thereby render the `reviewThreshold` useless.

## Vulnerability Detail

In `QVBaseStrategy` to ensure additional safety, multiple poolManagers need to review one applicant before he can be accepted. The number of revies to a particular recommendation (accepted or rejected ) must be equal or exceed the set `reviewThreshold`. This safeguard it in place to make sure that multiple poolManager take a look at the same recipient. The problem arises from the fact that it is not tracked if a manager already reviewed the recipient or not. This can lead to the situation where the same manager reviews one recipient multiple times and the recipient is thereby accepted or rejected even though not the required number of poolManagers have revied the recipient.

Example:

The `reviewThreshold` for a `QVBaseStrategy` is set to 2. Alice, a pool manager, reviews one batch of recipients and reviews Bob, a potential recipient, as accepted. When reviewing a second batch of recipients she accidently adds the review of Bob to the data again and reviews Bob as accepted again. Now Bob was accepted even though only one poolManager reviewed him an not the recommended two.


## Impact
Recipients are accepted/rejected even though they have not been reviewed by the recommended number of poolManagern

## Code Snippet

https://github.com/sherlock-audit/2023-09-Gitcoin/blob/6430c8004017e96ae2f5aac365bdefd0b6eeea72/allo-v2/contracts/strategies/qv-base/QVBaseStrategy.sol#L254-L288

## Tool used

Manual Review

## Recommendation

Track who revied which applicant to make sure that no poolManager reviews an applicant multiple times and thereby bypasses the safety guard of the `reviewThreshold`
