Jumpy Pear Beaver

high

# If an attacker can cause the `claim#recipientAddress`  to be blocklisted (usdc), their funds will be stuck
## Summary
An attacker can transfer either a few blocklisted tokens to the contract/recipient 
## Vulnerability Detail
since usdc can blocklist any address that is receiving or sending tokens 
an attacker blocklist the contract  and it will revert
ex:
Alice gives 500 Usdc `claim` to jake 
John gives 500 Usdc `claim` to Anthony 
The attacker gives little blocklisted funds to the contract 
Jake tries to claim their funds but its stuck and reverts 
Anthony tries to claim and it reverts  
https://github.com/d-xo/weird-erc20#tokens-with-blocklists
## Impact
If the contract gets blocklisted then all users who want to claim cant 
if the recipients get blocklisted individually, their claim is also going to revert too
Since `poolAmount` is not accounted for here the funds cant be rescued 
https://github.com/sherlock-audit/2023-09-Gitcoin/blob/main/allo-v2/contracts/strategies/donation-voting-merkle-distribution-vault/DonationVotingMerkleDistributionVaultStrategy.sol#L107
## Code Snippet
```solidity
// @auditor as we can see poolAmount is not updated or removed 
  function _afterAllocate(bytes memory _data, address _sender) internal override {
        // Decode the '_data' to get the recipientId, amount, and token
        (address recipientId, Permit2Data memory p2Data) = abi.decode(_data, (address, Permit2Data));

        // Get the token address
        address token = p2Data.permit.permitted.token;
        uint256 amount = p2Data.permit.permitted.amount;
            PERMIT2.permitTransferFrom(
                // The permit message.
                p2Data.permit,
                // The transfer recipient and amount.
                ISignatureTransfer.SignatureTransferDetails({to: address(this), requestedAmount: amount}),
                // Owner of the tokens and signer of the message.
                _sender,
                // The packed signature that was the result of signing
                // the EIP712 hash of `_permit`.
                p2Data.signature
            );
        }

        // Update the total payout amount for the claim
        claims[recipientId][token] += amount;
    }
}
```
```solidity
 function claim(Claim[] calldata _claims) external nonReentrant onlyAfterAllocation {
        uint256 claimsLength = _claims.length;

        // Loop through the claims
        for (uint256 i; i < claimsLength;) {
            Claim memory singleClaim = _claims[i];
            Recipient memory recipient = _recipients[singleClaim.recipientId];
            uint256 amount = claims[singleClaim.recipientId][singleClaim.token];

            // If the claim amount is zero this will revert
            if (amount == 0) {
                revert INVALID();
            }

            /// Delete the claim from the mapping
            delete claims[singleClaim.recipientId][singleClaim.token];

            address token = singleClaim.token;

            // Transfer the tokens to the recipient
           // @audit a usdc token will revert since this contract is blocklisted 
            _transferAmount(token, recipient.recipientAddress, amount);
```
## Tool used

Manual Review

## Recommendation
The best way to fix this on the contract side is to have a way to remove funds before the blocklisted tokens but I don't recommend this.
Another thing you can do is contact the right authorities about the blocklisted tokens.
If the recipient address gets blocklisted have a way after allocation to change their address effectively  using pull over push

