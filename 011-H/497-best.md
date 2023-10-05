Brief Mahogany Tiger

high

# Malicious registrant can front-run `RFPSimpleStrategy._allocate()` in order to change the `proposalBid` and get a bigger payout in the distribution

The `RFPSimpleStrategy::_allocate()` function can be frontrun by a malicious `registrant` chainging the `proposalBid` and get a bigger payout in the `RFPSimpleStrategy::_distribute()` function.

## Vulnerability Detail

Users can register to the pool strategy using the [RFPSimpleStrategy::_registerRecipient()](https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/rfp-simple/RFPSimpleStrategy.sol#L314C14-L314C32) function specifying the [proposalBid](https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/rfp-simple/RFPSimpleStrategy.sol#L378) in the registration. Then the pool manager [accepts the `registrant recipient`](https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/rfp-simple/RFPSimpleStrategy.sol#L404) using the [RFPSimpleStrategy::_allocate()](https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/rfp-simple/RFPSimpleStrategy.sol#L386) function.

The problem is that the execution of the `RFPSimpleStrategy::_allocate()` function by the `pool manager` can be frontun by a malicious `registrant recipient`. Consider the next scenario:

1. `UserA` call the `RFPSimpleStrategy::_registerRecipient()` using a `proposalBid=10`.
2. Pool manager accepts the proposal by `UserA` and call the `RFPSimpleStrategy::_allocate()` function.
3. `UserA` monitors the mempool and frontrun the manager `_allocate()` execution changing the proposal now `proposalBid=50`.
4. The `step 2` call finally is executed but the using non-agreed proposal `proposalBid=50`.

Now the `UserA` is accepted registrant recipient with non-agreed proposal bid (`proposalBid=50`).

## Impact

Malicious registrant can change the `proposalBid` to a non-agreed term causing that he can receive a bigger payout in the [RFPSimpleStrategy::_distribute()](https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/rfp-simple/RFPSimpleStrategy.sol#L417) function because in the [code line 435](https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/rfp-simple/RFPSimpleStrategy.sol#L435) the `proposalBid` is used to calculate the amount to pay to the `accepted registrant recipient`:

```solidity
File: RFPSimpleStrategy.sol
417:     function _distribute(address[] memory, bytes memory, address _sender)
418:         internal
419:         virtual
420:         override
421:         onlyInactivePool
422:         onlyPoolManager(_sender)
423:     {
...
...
433: 
434:         // Calculate the amount to be distributed for the milestone
435:         uint256 amount = (recipient.proposalBid * milestone.amountPercentage) / 1e18;
436: 
437:         // Get the pool, subtract the amount and transfer to the recipient
438:         poolAmount -= amount;
439:         _transferAmount(pool.token, recipient.recipientAddress, amount);
...
...
450:     }
```

The malicious accepted registrant can drain all funds from the pool strategy using one milestone.

## Code Snippet

- [RFPSimpleStrategy::_registerRecipient()](https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/rfp-simple/RFPSimpleStrategy.sol#L314C14-L314C32)
- [RFPSimpleStrategy::_allocate()](https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/rfp-simple/RFPSimpleStrategy.sol#L386)
- [RFPSimpleStrategy::_distribute()](https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/rfp-simple/RFPSimpleStrategy.sol#L417)

## Tool used

Manual review

## Recommendation

Verify the `proposalBid` when the `_allocate()` occurs:

```diff
    function _allocate(bytes memory _data, address _sender)
        internal
        virtual
        override
        nonReentrant
        onlyActivePool
        onlyPoolManager(_sender)
    {
        // Decode the '_data'
--      acceptedRecipientId = abi.decode(_data, (address));
++      (acceptedRecipientId, uint256 expectedProposalBid) = abi.decode(_data, (address, uint256));

        Recipient storage recipient = _recipients[acceptedRecipientId];

--      if (acceptedRecipientId == address(0) || recipient.recipientStatus != Status.Pending) {
++      if (acceptedRecipientId == address(0) || recipient.recipientStatus != Status.Pending || recipient.proposalBid != expectedProposalBid) {
            revert RECIPIENT_ERROR(acceptedRecipientId);
        }

        // Update status of acceptedRecipientId to accepted
        recipient.recipientStatus = Status.Accepted;

        _setPoolActive(false);

        IAllo.Pool memory pool = allo.getPool(poolId);

        // Emit event for the allocation
        emit Allocated(acceptedRecipientId, recipient.proposalBid, pool.token, _sender);
    }
```