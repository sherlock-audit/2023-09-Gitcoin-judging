Merry Punch Caterpillar

medium

# Anchor creation and cloneable strategies do not work on zkSync Era
## Summary

The contract states that it will run on zkSync Era. However, zkSync Era requires that the bytecode be statically known when CREATE or CREATE2 is used. This protocol has multiple places where it deploys a contract whose bytecode is not statically known

There is a similar issue submitted to Kyberswap; presumably, this should be judged the same way.

## Vulnerability Detail

See summary  and code snippet

## Impact

Will not work on zkSync Era

## Code Snippet

Uses of libraries that deploy:

https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/core/Registry.sol#L350
https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/core/Allo.sol#L190

Observing the source of both libraries shows that they modify the bytecode before deploying.

## Tool used

Manual Review

## Recommendation

Don't support zkSync Era