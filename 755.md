Jumpy Pear Beaver

medium

# when weird token reverts on zero transfer  which can cause a dos attack in `_distribute#RFPCommittee`
## Summary
A large  threshold RFP committee can cause a dos in `_distribute` by actions that could happen from a weird ERC20 token
## Vulnerability Detail
When the strategy is in a state of distributing
 the milestone should be paid out to the `accptedRecipeint`
If it [results](https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/rfp-simple/RFPSimpleStrategy.sol#L435) in `amount = 0` by it being ` (1 * 1e17 /1e18) `. 
:ex the `Lend` [token](https://github.com/d-xo/weird-erc20#revert-on-zero-value-transfers) will revert not allowing further [milestones](https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/rfp-simple/RFPSimpleStrategy.sol#L417) to be called. 
To fix this the pool manager would have to come in  and make the `poolActive` and `_allocate` again on a different recipient if they  have a large threshold and PoolManagers are multi sigs it will take more than 1 year to resolve causing the dos attack according to sherlock rules 
## Impact
The reason why its medium issue is because
>There is a viable scenario (even if unlikely) that could cause the protocol to enter a state where a material amount of funds can be lost. The attack path is possible with assumptions that either mimic on-chain conditions or reflect conditions that have a reasonable chance of becoming true in the future. 
1. it's unlikely state  a. that it that token reverts like above  b. the threshold is very large and this exploit requires that most pool managers are slow  because they are multisgs and a lot of them that are not ready to manage  in the most efficient way
2. it can be considered to be true in the future  and if pool managers grow which is very likely 
## Code Snippet
```solidity
//_distribute
   function _distribute(address[] memory, bytes memory, address _sender)
        internal
        virtual
        override
        onlyInactivePool
        onlyPoolManager(_sender)
    {
   if (recipient.proposalBid > poolAmount) revert NOT_ENOUGH_FUNDS();

        // Calculate the amount to be distributed for the milestone
       // @audit as we can see  0 is possible 
        uint256 amount = (recipient.proposalBid * milestone.amountPercentage) / 1e18;

        // Get the pool, subtract the amount and transfer to the recipient
        poolAmount -= amount;
        _transferAmount(pool.token, recipient.recipientAddress, amount);
        
        // Set the milestone status to 'Accepted'
        milestone.milestoneStatus = Status.Accepted;

        // Increment the upcoming milestone
        upcomingMilestone++;
```
```solidity
// committee voting 
function _allocate(bytes memory _data, address _sender) internal override onlyPoolManager(_sender) {
        if (acceptedRecipientId != address(0)) {
            revert RECIPIENT_ALREADY_ACCEPTED();
        }
       if (votes[recipientId] == voteThreshold) {
            // Set the accepted recipient
            acceptedRecipientId = recipientId;

            Recipient storage recipient = _recipients[acceptedRecipientId];
            recipient.recipientStatus = Status.Accepted;

            // Set the pool to inactive
            _setPoolActive(false);
```
## Tool used

Manual Review

## Recommendation
to not make zero transfer revert on a random token
```solidity
// note pseudo code 
if(amount)==0{
upcomingMilestones++;
}
```
I think the best way to handle this so to just `++Milestone` since dusting errors are excepted and shouldn't cause big dos issues.