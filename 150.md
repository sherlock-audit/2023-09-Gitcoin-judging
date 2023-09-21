Clever Metal Giraffe

high

# `QVSimpleStrategy` never updates `allocator.voiceCredits`.
## Summary

Every allocator in `QVSimpleStrategy` has a maximum credit limit. An allocator should not be able to bypass the limit. However, `QVSimpleStrategy` fails to record the allocated votes. An allocator can vote as many as possible.

## Vulnerability Detail

`QVSimpleStrategy._allocate` calls `_hasVoiceCreditsLeft` to check that the recipient has voice credits left to allocate.
https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/qv-simple/QVSimpleStrategy.sol#L121
```solidity
    function _allocate(bytes memory _data, address _sender) internal virtual override {
        …

        // check that the recipient has voice credits left to allocate
        if (!_hasVoiceCreditsLeft(voiceCreditsToAllocate, allocator.voiceCredits)) revert INVALID();

        _qv_allocate(allocator, recipient, recipientId, voiceCreditsToAllocate, _sender);
    }
```

`QVSimpleStrategy._hasVoiceCreditsLeft` checks ` _voiceCreditsToAllocate + _allocatedVoiceCredits <= maxVoiceCreditsPerAllocator`
https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/qv-simple/QVSimpleStrategy.sol#L144
```solidity
    function _hasVoiceCreditsLeft(uint256 _voiceCreditsToAllocate, uint256 _allocatedVoiceCredits)
        internal
        view
        override
        returns (bool)
    {
        return _voiceCreditsToAllocate + _allocatedVoiceCredits <= maxVoiceCreditsPerAllocator;
    }
```

The problem is that `allocator.voiceCredits` is always zero. Both `QVSimpleStrategy` and `QVBaseStrategy` don't update `allocator.voiceCredits`. Thus, allocators can cast more votes than `maxVoiceCreditsPerAllocator`.

## Impact

Every allocator has an unlimited number of votes.

## Code Snippet

https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/qv-simple/QVSimpleStrategy.sol#L121
https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/qv-simple/QVSimpleStrategy.sol#L144


## Tool used

Manual Review

## Recommendation

Updates `allocator.voiceCredits` in  `QVSimpleStrategy._allocate`.

```diff
    function _allocate(bytes memory _data, address _sender) internal virtual override {
        …

        // check that the recipient has voice credits left to allocate
        if (!_hasVoiceCreditsLeft(voiceCreditsToAllocate, allocator.voiceCredits)) revert INVALID();
+       allocator.voiceCredits += voiceCreditsToAllocate;
        _qv_allocate(allocator, recipient, recipientId, voiceCreditsToAllocate, _sender);
    }
```
