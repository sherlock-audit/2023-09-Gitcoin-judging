Rhythmic Rusty Swan

medium

# ```_createPool``` reverts even if the ```msg.value``` == ```baseFee```
## Summary
Upon creation of a pool a ```baseFee``` must be paid through the ```msg.value```. If an amount is also being deposited alongside the creation of the pool, then the ```msg.value``` must be >= ```baseFee``` + ```_amount```.

## Vulnerability Detail
However the transaction reverts for the case where ```msg.value == baseFee``` or ```msg.value ==baseFee + _amount``` stopping the pool from being created, when in reality it should've been created.  

## Impact
Even though the ```msg.value``` of the transaction was sufficient to create the pool, the transaction is still reverted. This can lead to a loss of funds in the contract as an excess is being provided in order to fulfil the check statement. 

## Code Snippet

https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/core/Allo.sol#L473-L475

## Tool used

Manual Review

## Recommendation
As the pool should be created for the instance when ```msg.value == baseFee + _amount``` or ```msg.value == baseFee``, we should change the if statement to:

``` if ((_token == NATIVE && (baseFee + _amount > msg.value)) || (_token != NATIVE && baseFee > msg.value))```

meaning we should remove the equals signs from the check so that the ```_createPool``` function doesn't revert under this instance.