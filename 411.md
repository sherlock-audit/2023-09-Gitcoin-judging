Shiny Gingham Bee

high

# Can not create a pool by cloning strategies on zkSync network
## Summary
Can not create pool by cloning strategies on zkSync network because of different behaviors from EVM instructions between zkSync and Ethereum

## Vulnerability Detail
When creating pool by cloning strategies, logics inside `Clone::createClone(address,uint256)` is executed, which calls to `ClonesUpgradeable::cloneDeterministic()`. Here, OpenZeppelin implements `ClonesUpgradeable::cloneDeterministic()` to deploy minimal proxy using almost assembly and using `create2`. 

So far, cloning a strategy using library `Clone` would get failed as [zksync docs pointed out](https://era.zksync.io/docs/reference/architecture/differences-with-ethereum.html#evm-instructions)

PoC:
[Check this deployed address on zksync era testnet](https://goerli.explorer.zksync.io/address/0xB1ae3211B3171a866bDCC2a69dA36448F594B9F6#contract). Try clone(), we would get failed

## Impact
Protocol does not work properly on zksync era: could not create pool by cloning strategies

## Code Snippet
https://github.com/allo-protocol/allo-v2/blob/851571c27df5c16f6586ece2a1cb6fd0acf04ec9/contracts/core/Allo.sol#L190
https://github.com/allo-protocol/allo-v2/blob/851571c27df5c16f6586ece2a1cb6fd0acf04ec9/contracts/core/libraries/Clone.sol#L26-L35

## Tool used

Manual Review

## Recommendation
Consider implementing different pattern for deploying/cloning strategies, such as: Using Factory for each kind of strategy and allow `Allo` contract to calls to Factory to deploy/clone strategies