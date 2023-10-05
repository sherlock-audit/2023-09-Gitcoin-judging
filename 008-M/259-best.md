Little Cloth Coyote

medium

# Incorrect validation in `submitUpcomingMilestone()`
Authorization checks for milestone submissions are not implemented correctly.

## Vulnerability Detail
According to the `submitUpcomingMilestones()` NatSpec, it's expected that `msg.sender` should be both the `acceptedRecipientId` AND a profile member. However, the current implementation does not enforce this correctly. It allows individuals who are not profile members but are `acceptedRecipientId` to call this function. Conversely, it also permits profile members who are not `acceptedRecipientId` to bypass this check and submit milestones on behalf of `acceptedRecipientId`. This behavior deviates from the intended functionality and may lead to unauthorized milestone submissions.

```solidity
    /// @notice Submit milestone by the acceptedRecipientId.
    /// @dev 'msg.sender' must be the 'acceptedRecipientId' and must be a member
    ///      of a 'Profile' to sumbit a milestone. Emits a 'MilestonesSubmitted()' event.
    /// @param _metadata The proof of work
    function submitUpcomingMilestone(Metadata calldata _metadata) external {
        // Check if the 'msg.sender' is the 'acceptedRecipientId' and is a member of the 'Profile'
        if (acceptedRecipientId != msg.sender && !_isProfileMember(acceptedRecipientId, msg.sender)) {
            revert UNAUTHORIZED();
        }

        // Check if the upcoming milestone is in fact upcoming
        if (upcomingMilestone >= milestones.length) revert INVALID_MILESTONE();

        // Get the milestone and update the metadata and status
        Milestone storage milestone = milestones[upcomingMilestone];
        milestone.metadata = _metadata;

        // Set the milestone status to 'Pending' to indicate that the milestone is submitted
        milestone.milestoneStatus = Status.Pending;

        // Emit event for the milestone
        emit MilstoneSubmitted(upcomingMilestone);
    }

```
## Impact
Unauthorized individual can submit milestones.
## Code Snippet
https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/rfp-simple/RFPSimpleStrategy.sol#L253
## Tool used

Manual Review

## Recommendation
```solidity
    function submitUpcomingMilestone(Metadata calldata _metadata) external {
        // Check if the 'msg.sender' is the 'acceptedRecipientId' and is a member of the 'Profile'
-       if (acceptedRecipientId != msg.sender && !_isProfileMember(acceptedRecipientId, msg.sender)) {
+       if (acceptedRecipientId != msg.sender ||  !_isProfileMember(acceptedRecipientId, msg.sender)) {
            revert UNAUTHORIZED();
        }

        // Check if the upcoming milestone is in fact upcoming
        if (upcomingMilestone >= milestones.length) revert INVALID_MILESTONE();

        // Get the milestone and update the metadata and status
        Milestone storage milestone = milestones[upcomingMilestone];
        milestone.metadata = _metadata;

        // Set the milestone status to 'Pending' to indicate that the milestone is submitted
        milestone.milestoneStatus = Status.Pending;

        // Emit event for the milestone
        emit MilstoneSubmitted(upcomingMilestone);
    }
```