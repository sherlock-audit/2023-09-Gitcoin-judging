Brief Mahogany Tiger

medium

# The `RFPSimpleStrategy._registerRecipient()` does not work when the strategy was created using the `useRegistryAnchor=true` causing that nobody can register to the pool

The [RFPSimpleStrategy._registerRecipient()](https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/rfp-simple/RFPSimpleStrategy.sol#L314) does not work when the strategy was created using the `useRegistryAnchor=true` causing that no one can register to the pool and the funds sent to the pool may be trapped.

## Vulnerability Detail

The `RFPSimpleStrategy` strategies can be created using the `useRegistryAnchor` which indicates whether to use the registry anchor or not. If the pool is created using the `useRegistryAnchor=true` the [RFPSimpleStrategy._registerRecipient()](https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/rfp-simple/RFPSimpleStrategy.sol#L314) will be reverted by [RECIPIENT_ERROR](https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/rfp-simple/RFPSimpleStrategy.sol#L362C52-L362C67). The problem is that when `useRegistryAnchor` is true, the variable [recipientAddress is not collected](https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/rfp-simple/RFPSimpleStrategy.sol#L329-L332) so the function will revert by the [RECIPIENT_ERROR](https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/rfp-simple/RFPSimpleStrategy.sol#L362).

I created a test where the strategy is created using the `userRegistryAnchor=true` then the `registerRecipient()` will be reverted by the `RECIPIENT_ERROR`.

```solidity
// File: test/foundry/strategies/RFPSimpleStrategy.t.sol:RFPSimpleStrategyTest
// $ forge test --match-test "test_registrationIsBlockedWhenThePoolIsCreatedWithUseRegistryIsTrue" -vvv
//
    function test_registrationIsBlockedWhenThePoolIsCreatedWithUseRegistryIsTrue() public {
        // The registerRecipient() function does not work then the strategy was created using the
        // useRegistryAnchor = true.
        //
        bool useRegistryAnchorTrue = true;
        RFPSimpleStrategy custom_strategy = new RFPSimpleStrategy(address(allo()), "RFPSimpleStrategy");

        vm.prank(pool_admin());
        poolId = allo().createPoolWithCustomStrategy(
            poolProfile_id(),
            address(custom_strategy),
            abi.encode(maxBid, useRegistryAnchorTrue, metadataRequired),
            NATIVE,
            0,
            poolMetadata,
            pool_managers()
        );
        //
        // Create profile1 metadata and anchor
        Metadata memory metadata = Metadata({protocol: 1, pointer: "metadata"});
        address anchor = profile1_anchor();
        bytes memory data = abi.encode(anchor, 1e18, metadata);
        //
        // Profile1 member registers to the pool but it reverted by RECIPIENT_ERROR
        vm.startPrank(address(profile1_member1()));
        vm.expectRevert(abi.encodeWithSelector(RECIPIENT_ERROR.selector, address(anchor)));
        allo().registerRecipient(poolId, data);
    }
```

## Impact

The pool created with a strategy using the `userRegistryAnchor=true` can not get registrants because `_registerRecipient()` will be reverted all the time. If the pool is funded but no one can be allocated since [there is not registered recipients](https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/rfp-simple/RFPSimpleStrategy.sol#L397), the deposited funds by others may be trapped because those are not [distributed](https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/rfp-simple/RFPSimpleStrategy.sol#L429) since there are not `registrants`.


## Code Snippet

- [_registerRecipient()](https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/rfp-simple/RFPSimpleStrategy.sol#L314)

## Tool used

Manual review

## Recommendation

When the strategy is using `useRegistryAncho=true`, get the `recipientAddress` from the `data`:

```diff
    function _registerRecipient(bytes memory _data, address _sender)
        internal
        override
        onlyActivePool
        returns (address recipientId)
    {
        bool isUsingRegistryAnchor;
        address recipientAddress;
        address registryAnchor;
        uint256 proposalBid;
        Metadata memory metadata;

        // Decode '_data' depending on the 'useRegistryAnchor' flag
        if (useRegistryAnchor) {
            /// @custom:data when 'true' -> (address recipientId, uint256 proposalBid, Metadata metadata)
--          (recipientId, proposalBid, metadata) = abi.decode(_data, (address, uint256, Metadata));
++          (recipientId, recipientAddress, proposalBid, metadata) = abi.decode(_data, (address, address, uint256, Metadata));

            // If the sender is not a profile member this will revert
            if (!_isProfileMember(recipientId, _sender)) revert UNAUTHORIZED();
```