Atomic Ultraviolet Mole

medium

# ````reviewRecipients()```` might rollback recipients' new applications
Due to the asynchronous nature of blockchain, there is a time delay from when a recipient initiates a registration transaction, to when that transaction is mined, and finally to when it is displayed on pool manager's frontend. But ````reviewRecipients()```` of ````DonationVotingMerkleDistributionBaseStrategy```` update ````statusesBitMap```` in ````fullRow```` mode without checking if new applications submitted during this ````delay````. This would cause new applications to be overwritten from ````Status.Pending```` to ````Status.None```` .

## Vulnerability Detail
Let's look at the implementation of ````reviewRecipients()````, ````statusesBitMap```` is updated with ````fullRow```` without checking data freshness.
```solidity
File: contracts\strategies\donation-voting-merkle-base\DonationVotingMerkleDistributionBaseStrategy.sol
341:     function reviewRecipients(ApplicationStatus[] memory statuses)
342:         external
343:         onlyActiveRegistration
344:         onlyPoolManager(msg.sender)
345:     {
346:         // Loop through the statuses and set the status
347:         for (uint256 i; i < statuses.length;) {
348:             uint256 rowIndex = statuses[i].index;
349:             uint256 fullRow = statuses[i].statusRow;
350: 
351:             statusesBitMap[rowIndex] = fullRow;
352: 
353:             // Emit that the recipient status has been updated with the values
354:             emit RecipientStatusUpdated(rowIndex, fullRow, msg.sender);
355: 
356:             unchecked {
357:                 i++;
358:             }
359:         }
360:     }


```

Now, let's consider the case below:
_for simplicity, let a ````fullRow=0xABCD```` represents 4 recipients' status, 4 bits per recipient._
(1) pool manager begins to review applications and there are two pending applications, the initial states are
```solidity
block.timestamp = 100
fullRow = 0x0011
```

(2) a third recipient's application initiated, but doesn't mined
```solidity
block.timestamp = 195
fullRow = 0x0011
```
(3) the pool manager approves the first two applications, initiate a transaction with new ````fullRow = 0x0022````
```solidity
block.timestamp = 200
fullRow = 0x0011
```

(4) the third recipient's application transaction mined
```solidity
block.timestamp = 205
fullRow = 0x0111
```

(5) the pool manager's approval transaction mined
```solidity
block.timestamp = 210
fullRow = 0x0022
```
We can see the third application's status is overwritten from ````Status.Pending(1)```` to ````Status.None(0)```` .

## Impact
(1) recipients' applications might be rolled back
(2) updates from other pool managers might also be overwritten.

## Code Snippet
https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/donation-voting-merkle-base/DonationVotingMerkleDistributionBaseStrategy.sol#L341

## Tool used

Manual Review

## Recommendation
P.S. this solution is easy but can't address the conflicts between pool managers.
```diff
File: contracts\strategies\donation-voting-merkle-base\DonationVotingMerkleDistributionBaseStrategy.sol
-341:     function reviewRecipients(ApplicationStatus[] memory statuses)
+341:     function reviewRecipients(ApplicationStatus[] memory statuses, uint256 refRecipientsCounter)
342:         external
343:         onlyActiveRegistration
344:         onlyPoolManager(msg.sender)
345:     {
+            if (refRecipientsCounter != recipientsCounter) revert STATUSES_OUTDATED();
346:         // Loop through the statuses and set the status
347:         for (uint256 i; i < statuses.length;) {
348:             uint256 rowIndex = statuses[i].index;
349:             uint256 fullRow = statuses[i].statusRow;
350: 
351:             statusesBitMap[rowIndex] = fullRow;
352: 
353:             // Emit that the recipient status has been updated with the values
354:             emit RecipientStatusUpdated(rowIndex, fullRow, msg.sender);
355: 
356:             unchecked {
357:                 i++;
358:             }
359:         }
360:     }


```