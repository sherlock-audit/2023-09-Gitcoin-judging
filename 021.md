Fancy Khaki Perch

medium

# `Anchor.execute` can only be executed in two separate transactions.
## Summary
`Anchor.execute` can only be executed in two separate transactions.
## Vulnerability Detail
`Anchor.execute` is not a `payable` function. Therefore, if a user wants to send native tokens directly through `Anchor.execute`, he must first send the native tokens to the `receive` function in a separate transaction.

https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/core/Anchor.sol#L70-L87
```solidity
    function execute(address _target, uint256 _value, bytes memory _data) external returns (bytes memory) {
        // Check if the caller is the owner of the profile and revert if not
        if (!registry.isOwnerOfProfile(profileId, msg.sender)) revert UNAUTHORIZED();

        // Check if the target address is the zero address and revert if it is
        if (_target == address(0)) revert CALL_FAILED();

        // Call the target address and return the data
        (bool success, bytes memory data) = _target.call{value: _value}(_data);

        // Check if the call was successful and revert if not
        if (!success) revert CALL_FAILED();

        return data;
    }

    /// @notice This contract should be able to receive native token
    receive() external payable {}
```

The protocol doesn't restrict users to having a pre-existing balance in Anchor before executing `Anchor.execute`. The current design compromises the integrity of the protocol's functionality, requiring users to make two separate transactions to achieve a single execute operation.

Additionally, the failure in the test `test_execute_CALL_FAILED` is not due to the `NoFallbackContract`, but rather due to insufficient native tokens. The error code is `EvmError: OutOfFund`, which is precisely the impact of this vulnerability.

```shell
forge test --mt 'test_execute_CALL_FAILED' -vvvv
[⠒] Compiling...
No files changed, compilation skipped

Running 2 tests for test/foundry/core/Anchor.t.sol:AnchorTest
[PASS] test_execute_CALL_FAILED() (gas: 93744)
Traces:
  [93744] AnchorTest::test_execute_CALL_FAILED()
    ├─ [22509] MockRegistry::setOwnerOfProfile(0x746573745f70726f66696c650000000000000000000000000000000000000000, AnchorTest: [0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496])
    │   └─ ← ()
    ├─ [12666] → new NoFallbackContract@0x2e234DAe75C793f67A35089C9d99245E1C58470b
    │   └─ ← 63 bytes of code
    ├─ [0] VM::expectRevert(CALL_FAILED())
    │   └─ ← ()
    ├─ [8834] Anchor::execute(NoFallbackContract: [0x2e234DAe75C793f67A35089C9d99245E1C58470b], 1000000000000000000 [1e18], 0x35b09a6e)
    │   ├─ [527] MockRegistry::isOwnerOfProfile(0x746573745f70726f66696c650000000000000000000000000000000000000000, AnchorTest: [0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496]) [staticcall]
    │   │   └─ ← true
    │   ├─ [0] NoFallbackContract::someFunction{value: 1000000000000000000}()
    │   │   └─ ← "EvmError: OutOfFund"
    │   └─ ← "CALL_FAILED()"
    └─ ← ()
```

## Impact
The current design compromises the integrity of the protocol's functionality, requiring users to make two separate transactions to achieve a single execute operation.

A specific scenario is as follows:
1. The user sets `Anchor` as the recipient address for the pool and deposits 100 ETH through `Allo`.
2. The user wants to deposit 100 ETH into another fund contract through `Anchor`, and that contract will charge a 0.001 ETH fee.
3. The user wants to ensure that there is exactly 100 ETH in the fund contract, so he needs to send a total of 100.001 ETH.
4. Because the `execute` function is not `payable`, he first needs to send 0.001 ETH to the `receive` function, and then execute the `execute` function.
5. Splitting it into two transactions creates a poor user experience, making the user pay an additional gas fee, which could even exceed the extra 0.001 ETH he intended to send.
## Code Snippet
https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/core/Anchor.sol#L70-L87
## Tool used

Manual Review

## Recommendation
Add `payable` to `execute`.