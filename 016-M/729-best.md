Bumpy Charcoal Squid

medium

# `QVBaseStrategy` contract : recipient `reviewStatus` is not reset upon re-registration

`QVBaseStrategy` contract : the reviewStatus of the recipient is not reset (set to zero) when he re-registres again.

## Vulnerability Detail

- In `QVBaseStrategy` strategy contract: when the user first registers; his status is updated from `None` to `Pending`.

- Then the pool manager can review recipients (via `reviewRecipients` function) and update their statuses from `Pending` to `Accepted` and from `Accepted` to `Pending` only (not updating to `Rejected` or `Appealed`) if these recipients statuses got votes equal to `reviewThreshold`:

  [QVBaseStrategy::reviewRecipients /L275-280](https://github.com/allo-protocol/allo-v2/blob/0b881ef4a0013d2809374c9ea69f4cf1288dfe62/contracts/strategies/qv-base/QVBaseStrategy.sol#L275-L280)

  ```solidity
              if (reviewsByStatus[recipientId][recipientStatus] >= reviewThreshold) {
                  Recipient storage recipient = recipients[recipientId];
                  recipient.recipientStatus = recipientStatus;

                  emit RecipientStatusUpdated(recipientId, recipientStatus, address(0));
              }
  ```

- The strategy contract allows registered recipients to re-register again with new terms; and when doing so, their statuses are updated from `Accepted` ==> `Pending` or from `Rejected` to `Appealed`:

  [QVBaseStrategy::\_registerRecipient /L275-280](https://github.com/allo-protocol/allo-v2/blob/0b881ef4a0013d2809374c9ea69f4cf1288dfe62/contracts/strategies/qv-base/QVBaseStrategy.sol#L414-L429)

  ```solidity
  if (currentStatus == Status.None) {
            // recipient registering new application
            recipient.recipientStatus = Status.Pending;
            emit Registered(recipientId, _data, _sender);
        } else {
            if (currentStatus == Status.Accepted) {
                // recipient updating accepted application
                recipient.recipientStatus = Status.Pending;
            } else if (currentStatus == Status.Rejected) {
                // recipient updating rejected application
                recipient.recipientStatus = Status.Appealed;
            }

            // emit the new status with the '_data' that was passed in
            emit UpdatedRegistration(recipientId, _data, _sender, recipient.recipientStatus);
        }
  ```

## Impact

But as can be noticed; the reviewRecipient status of the re-registered recipient is not reset which will result in this re-registered recipient getting `Accepted` status on their new registration in the next review round/rounds with lesser votes to reach `reviewThreshold`.

## Code Snippet

[QVBaseStrategy::\_registerRecipient /L275-280](https://github.com/allo-protocol/allo-v2/blob/0b881ef4a0013d2809374c9ea69f4cf1288dfe62/contracts/strategies/qv-base/QVBaseStrategy.sol#L414-L429)

```solidity
if (currentStatus == Status.None) {
          // recipient registering new application
          recipient.recipientStatus = Status.Pending;
          emit Registered(recipientId, _data, _sender);
      } else {
          if (currentStatus == Status.Accepted) {
              // recipient updating accepted application
              recipient.recipientStatus = Status.Pending;
          } else if (currentStatus == Status.Rejected) {
              // recipient updating rejected application
              recipient.recipientStatus = Status.Appealed;
          }

          // emit the new status with the '_data' that was passed in
          emit UpdatedRegistration(recipientId, _data, _sender, recipient.recipientStatus);
      }
```

[QVBaseStrategy::reviewRecipients ](https://github.com/allo-protocol/allo-v2/blob/0b881ef4a0013d2809374c9ea69f4cf1288dfe62/contracts/strategies/qv-base/QVBaseStrategy.sol#L254-L288)

```solidity
  function reviewRecipients(address[] calldata _recipientIds, Status[] calldata _recipientStatuses)
      external
      virtual
      onlyPoolManager(msg.sender)
      onlyActiveRegistration
  {
      // make sure the arrays are the same length
      uint256 recipientLength = _recipientIds.length;
      if (recipientLength != _recipientStatuses.length) revert INVALID();

      for (uint256 i; i < recipientLength;) {
          Status recipientStatus = _recipientStatuses[i];
          address recipientId = _recipientIds[i];

          // if the status is none or appealed then revert
          if (recipientStatus == Status.None || recipientStatus == Status.Appealed) {
              revert RECIPIENT_ERROR(recipientId);
          }

          reviewsByStatus[recipientId][recipientStatus]++;

          if (reviewsByStatus[recipientId][recipientStatus] >= reviewThreshold) {
              Recipient storage recipient = recipients[recipientId];
              recipient.recipientStatus = recipientStatus;

              emit RecipientStatusUpdated(recipientId, recipientStatus, address(0));
          }

          emit Reviewed(recipientId, recipientStatus, msg.sender);

          unchecked {
              ++i;
          }
      }
  }
```

## Tool used

Manual Review

## Recommendation

Update `_registerRecipient` function to reset `reviewsByStatus[recipientId][recipientStatus]` when the recipient re-registers:

```diff
if (currentStatus == Status.None) {
          // recipient registering new application
          recipient.recipientStatus = Status.Pending;
          emit Registered(recipientId, _data, _sender);
      } else {
          if (currentStatus == Status.Accepted) {
              // recipient updating accepted application
              recipient.recipientStatus = Status.Pending;
+             reviewsByStatus[recipientId][Status.Accepted]=0;
          } else if (currentStatus == Status.Rejected) {
              // recipient updating rejected application
              recipient.recipientStatus = Status.Appealed;
          }

          // emit the new status with the '_data' that was passed in
          emit UpdatedRegistration(recipientId, _data, _sender, recipient.recipientStatus);
      }
```