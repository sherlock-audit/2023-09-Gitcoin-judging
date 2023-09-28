Clever Metal Giraffe

high

# `recipientsCounter` should start from 1 in `DonationVotingMerkleDistributionBaseStrategy`
## Summary

When doing `DonationVotingMerkleDistributionBaseStrategy._registerRecipient`, it checks the current status of the recipient. If the recipient is new to the pool, the status should be `Status.None`. However, `recipientsCounter` starts from 0. The new recipient actually gets the status of first recipient of the pool.

## Vulnerability Detail

`DonationVotingMerkleDistributionBaseStrategy._registerRecipient` calls `_getUintRecipientStatus` to get the current status of the application. The status of the new application should be `Status.None`. Then, the `recipientToStatusIndexes[recipientId]`  to `recipientsCounter` and `recipientsCounter`.
https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/donation-voting-merkle-base/DonationVotingMerkleDistributionBaseStrategy.sol#L580
```solidity
    function _registerRecipient(bytes memory _data, address _sender)
        internal
        override
        onlyActiveRegistration
        returns (address recipientId)
    {
        …

        uint8 currentStatus = _getUintRecipientStatus(recipientId);

        if (currentStatus == uint8(Status.None)) {
            // recipient registering new application
            recipientToStatusIndexes[recipientId] = recipientsCounter;
            _setRecipientStatus(recipientId, uint8(Status.Pending));

            bytes memory extendedData = abi.encode(_data, recipientsCounter);
            emit Registered(recipientId, extendedData, _sender);

            recipientsCounter++;
        } else {
            if (currentStatus == uint8(Status.Accepted)) {
                // recipient updating accepted application
                _setRecipientStatus(recipientId, uint8(Status.Pending));
            } else if (currentStatus == uint8(Status.Rejected)) {
                // recipient updating rejected application
                _setRecipientStatus(recipientId, uint8(Status.Appealed));
            }
            emit UpdatedRegistration(recipientId, _data, _sender, _getUintRecipientStatus(recipientId));
        }
    }
```

`DonationVotingMerkleDistributionBaseStrategy._getUintRecipientStatus` calls `_getStatusRowColumn` to get the column index and current row.
https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/donation-voting-merkle-base/DonationVotingMerkleDistributionBaseStrategy.sol#L819
```solidity
    function _getUintRecipientStatus(address _recipientId) internal view returns (uint8 status) {
        // Get the column index and current row
        (, uint256 colIndex, uint256 currentRow) = _getStatusRowColumn(_recipientId);

        // Get the status from the 'currentRow' shifting by the 'colIndex'
        status = uint8((currentRow >> colIndex) & 15);

        // Return the status
        return status;
    }
```

`DonationVotingMerkleDistributionBaseStrategy._getStatusRowColumn` computes indexes from `recipientToStatusIndexes[_recipientId]`. For the new recipient. Those indexes should be zero.
https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/donation-voting-merkle-base/DonationVotingMerkleDistributionBaseStrategy.sol#L833
```solidity
    function _getStatusRowColumn(address _recipientId) internal view returns (uint256, uint256, uint256) {
        uint256 recipientIndex = recipientToStatusIndexes[_recipientId];

        uint256 rowIndex = recipientIndex / 64; // 256 / 4
        uint256 colIndex = (recipientIndex % 64) * 4;

        return (rowIndex, colIndex, statusesBitMap[rowIndex]);
    }
```

The problem is that `recipientCounter` starts from zero.
https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/donation-voting-merkle-base/DonationVotingMerkleDistributionBaseStrategy.sol#L166
```solidity
    /// @notice The total number of recipients.
    uint256 public recipientsCounter;
```

Consider the following situation:
* Alice is the first recipient calls `registerRecipient`
```solidity
// in _registerRecipient
recipientToStatusIndexes[Alice] = recipientsCounter = 0;
_setRecipientStatus(Alice, uint8(Status.Pending));
recipientCounter++
```
* Bob calls `registerRecipient`.
```solidity
// in _getStatusRowColumn
recipientToStatusIndexes[Bob] = 0 // It would access the status of Alice
// in _registerRecipient
currentStatus = _getUintRecipientStatus(recipientId) = Status.Pending
currentStatus != uint8(Status.None) -> no new application is recorded in the pool.
```

This implementation error makes the pool can only record the first application.

## Impact

## Code Snippet

https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/donation-voting-merkle-base/DonationVotingMerkleDistributionBaseStrategy.sol#L580
https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/donation-voting-merkle-base/DonationVotingMerkleDistributionBaseStrategy.sol#L819
https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/donation-voting-merkle-base/DonationVotingMerkleDistributionBaseStrategy.sol#L833
https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/donation-voting-merkle-base/DonationVotingMerkleDistributionBaseStrategy.sol#L166

## Tool used

Manual Review

## Recommendation

Make the counter start from 1. There are two methods to fix the  issue.

1.
```diff
    /// @notice The total number of recipients.
+   uint256 public recipientsCounter;
-   uint256 public recipientsCounter;
```

2.
```diff
    function _registerRecipient(bytes memory _data, address _sender)
        internal
        override
        onlyActiveRegistration
        returns (address recipientId)
    {
        …

        uint8 currentStatus = _getUintRecipientStatus(recipientId);

        if (currentStatus == uint8(Status.None)) {
            // recipient registering new application
+           recipientToStatusIndexes[recipientId] = recipientsCounter + 1;
-           recipientToStatusIndexes[recipientId] = recipientsCounter;
            _setRecipientStatus(recipientId, uint8(Status.Pending));

            bytes memory extendedData = abi.encode(_data, recipientsCounter);
            emit Registered(recipientId, extendedData, _sender);

            recipientsCounter++;
        …
    }
```
