Mammoth Aquamarine Wallaby

medium

# M-04 Funds can be locked in the DonationVotingMerkleDistributionBaseStrategy due to having a receive payable function and the unique functionality of the withdraw function.
## Summary
Due to a payable receive function and the withdraw function only withdrawing based on internal accounting and tokens, funds can be stuck in the contract.
## Impact
Funds sent into the contract using the `receive` function, or other ERC20 tokens not of the `pool.token`  can remain locked in the contract.

## Vulnerability Detail
The internal accounting uses the `poolAmount` variable to keep track of the balances.

The receive function does not increment the `poolAmount` variable when funds are received, this in itself is not an issue as the contract token may not be the `NATIVE` token.

Two scenarios arise from this situation.
The first being that if the `pool.token` is ETH, the actual balance of the contract and the internal accounting can differ significantly.
The second aspect to keep in mind is that, should the `pool.token` not be ETH, the `poolAmount` variable is tracking the balance of the `pool.token`, and there is no mechanism to keep track of ETH funds that may be sent into the contract using the `receive` function.

In the first scenario the values would be misaligned leaving the funds locked in the contract and in the second scenario, the `NATIVE`(ETH) funds could never be withdrawn as the `withdraw` function uses the `pool.token` to decide what type of transfer to action.

Scenario1 (pool token is ETH):
1. The `poolAmount` is 1 ETH.
2. 10 ETH is sent in using the `receive` function.
3. The actual balance is now 11 ETH.
4. The pool admin withdraws 1 ETH.
5. The internal accounting decrements `poolAmount` to be ZERO.
6. The actual balance is now 10 ETH
7. The funds are locked because the withdraw function will revert due to any value sent into the withdraw function being greater the `poolAmount` (ZERO) as in the check below.
```solidity
if (_amount > poolAmount) {
            revert INVALID();
        }
```

Scenario2 (pool token is USDC):
1. The `poolAmount` is 1 USDC.
2. 10 ETH is sent in using the `receive` function.
3. The actual balance is now 1O ETH and 1 USDC.
4. The pool admin withdraws 1 USDC.
5. The internal accounting decrements `poolAmount` to be ZERO.
6. The actual balance is still 10 ETH
7. The funds are locked because the withdraw function will revert due to any value sent into the withdraw function being greater the `poolAmount` (ZERO) as in the check below.
```solidity
if (_amount > poolAmount) {
            revert INVALID();
        }
```

## Code Snippet
https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/donation-voting-merkle-base/DonationVotingMerkleDistributionBaseStrategy.sol#L394-L409

A similar scenario can be found in RFPSimpleStrategy.sol.

```solidity
    function withdraw(uint256 _amount) external onlyPoolManager(msg.sender) {
        if (block.timestamp <= allocationEndTime + 30 days) {
            revert INVALID();
        }

        IAllo.Pool memory pool = allo.getPool(poolId);

        if (_amount > poolAmount) {
            revert INVALID();
        }

        poolAmount -= _amount;

        // Transfer the tokens to the 'msg.sender' (pool manager calling function)
        _transferAmount(pool.token, msg.sender, _amount);
    }
```



## PoC
Copy/Paste this test function in the `DonationVotingMerkleDistributionBase.t.sol` file in the `test/foundry/strategies/` directory:
```solidity
function test_depositAndWithdraw() public {
        address tester = makeAddr("tester");
        vm.deal(tester, 100 ether);
        vm.startPrank(tester);
        console.log("[+] The strategy poolAmount variable value before deposit is : ",strategy.getPoolAmount());
        console.log("[+] The balance of the strategy contract before the call is : ",address(strategy).balance);
        console.log("[+] Now depositing 10 ETH");
        address(strategy).call{value: 10 ether}("");
        console.log("[+] The strategy poolAmount variable value after deposit is : ",strategy.getPoolAmount());
        console.log("[+] The balance of the strategy contract after the call is : ",address(strategy).balance);
        vm.stopPrank();
        vm.warp(block.timestamp + 90 days);
        console.log("[+] Now withdrawing 1 ETH");
        vm.startPrank(pool_admin());
        strategy.withdraw(1 ether);
        vm.stopPrank();
        console.log("[+] The strategy poolAmount variable value after withdrawal is : ",strategy.getPoolAmount());
        console.log("[+] The balance of the strategy contract after withdrawal is : ",address(strategy).balance);
        console.log("[+] There is still 10 ETH of funds to withdraw but the internal accounting shows ZERO");
        console.log("[+] Try withdrawing 1 ETH");
        console.log("[+] We are expecting a revert");
        vm.startPrank(pool_admin());
        vm.expectRevert();
        strategy.withdraw(1 ether);
        vm.stopPrank();
    }
```
## Output
```text
[PASS] test_depositAndWithdraw() (gas: 109514)
Logs:
  [+] The strategy poolAmount variable value before deposit is :  1000000000000000000
  [+] The balance of the strategy contract before the call is :  1000000000000000000
  [+] Now depositing 10 ETH
  [+] The strategy poolAmount variable value after deposit is :  1000000000000000000
  [+] The balance of the strategy contract after the call is :  11000000000000000000
  [+] Now withdrawing 1 ETH
  [+] The strategy poolAmount variable value after withdrawal is :  0
  [+] The balance of the strategy contract after withdrawal is :  10000000000000000000
  [+] There is still 10 ETH of funds to withdraw but the internal accounting shows ZERO
  [+] Try withdrawing 1 ETH
  [+] We are expecting a revert

Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 9.60ms
 
Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)
```
Run forge test 
```text
forge test --match-contract DonationVotingMerkleDistributionBaseMockTest --match-test test_depositAndWithdraw -vv
```
## Tool used

Manual Review

## Recommendation
Two possibilities could be considered.
1. Modify the withdraw function to work with the actual balance of the contract based on the token that is required to be withdrawn, and allow the caller to specify the token.
2. Increment the `poolAmount` on receiving funds via the `receive` function.

The second option would only work if the token would always be the `NATIVE` token, therefore it would be preferable to alter the withdraw function or implement a separate `sweep` function.