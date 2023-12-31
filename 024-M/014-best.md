Interesting Shamrock Lemur

high

# Token lockup vulnerability in _afterAllocate function

The `DonationVotingMerkleDistributionVaultStrategy.sol` smart contract is susceptible to a front-running attack during the execution of the `_afterAllocate` function, specifically when using the `permitTransferFrom` function from the Uniswap permit2 contract. This vulnerability arises because a malicious user can front-run the transaction, calling the `permitTransferFrom` function with the same parameters and the same signature. This action results in tokens being sent to the smart contract but without updating the total payout amount for the claim.
Furthermore, the user's transaction will subsequently revert due to protections on Uniswap permit2 contract against replay attacks, causing some user tokens to become permanently stuck within the `DonationVotingMerkleDistributionVaultStrategy.sol` contract.

## Vulnerability Detail

The vulnerability occurs when a user interacts with the `DonationVotingMerkleDistributionVaultStrategy.sol` contract and attempts to use the `permitTransferFrom` function from the Uniswap permit2 contract. A malicious user can see the transaction in the mempool and front-run the transaction. By doing so, he can call `permitTransferFrom` with the exact same parameters, leading to the transfer of tokens to the smart contract as desired. However, the total payout amount for the claim is not updated because the malicious user send tokens to the contract directly, and did not by the `_afterAllocate`. Additionally, the user's transaction would revert, because of the protection against replay attacks, resulting in some tokens being permanently locked within the smart contract.

Let's take an example, taking the [sequence diagram](https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/donation-voting-merkle-distribution-vault/README.md#sequence-diagram) to facilitate.

All the workflow is the same.

1. When Bob calls `allocate()`, the front-runner bot see it.
2. Then the front-runner is able to know each parameter of the call.
3. He converts this data, then he has:
- `p2Data.permit`
- `amount`
- `_sender`
- `p2Data.signature`
4. He calls the permit2 contract with following parameters: 
```solidity
    permitTransferFrom(
        p2Data.permit,
        ISignatureTransfer.SignatureTransferDetails({to: [STRATEGY_CONTRACT_ADDRESS], requestedAmount: amount}),
        _sender,
        p2Data.signature
    )
```
5. Tokens are well received.
6. Bob's transaction revert because the signature was already used.
7. `claims[recipientId][token] += amount` was not done. So `recipientId` will not be able to claim tokens.

Tokens are now frozen on the contract.

## Impact

The impact of this vulnerability is significant. A malicious user exploiting this issue can effectively lock tokens within the `DonationVotingMerkleDistributionVaultStrategy.sol` contract without properly updating the total payout amount for the claim. This can result in a discrepancy between the claimed and actual token balances, leading to potential financial losses for the affected users. Furthermore, the user's transaction reverting can further complicate the situation, making it impossible to recover the locked tokens.

## Code Snippet

https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/donation-voting-merkle-distribution-vault/DonationVotingMerkleDistributionVaultStrategy.sol#L121-L132

## Tool used

Manual Review

## Recommendation

To mitigate this vulnerability and prevent potential token lockups and discrepancies, it is recommended to revise the contract's logic. I recommend to delete the Uniswap permit2 feature and to only allow simple `transferFrom` calls.