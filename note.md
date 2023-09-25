001-L

Valid low, should be fixed. DoS could be avoided by sending extra 1wei of NATIVE.
#074, #101 provides demonstration of fund locking because of this and no refund,
but still low because only 1wei will be lost each call if to workaround this.

002-L

Informational. User' fault if he sends excessive token.

002-H
130.md, 246.md, 378.md provides a valid attack vector. The conditions is that the pool needs to hold overflowed fundings.

001-H
High. Unexpected fund loss. front run with register

003-M
useRegistryAnchor
Medium. Core function bricks with high possibility. No fund loss.

003-H
125.md
High. Fund loss because of infinite vote.

005-H
614,260 "also identified 003-H, should submit separatly"
High. voiceCreditsCastToRecipient Accounting error.

004-M
Arguable Medium. blacklist receiver.

005-M
zksync CREAT3 and clone

006-M
recipient.proposalBid > poolAmount revert


007-H
create3 anchor register address


008-H
merkle root empty

012-H
if (recipient.proposalBid > poolAmount) revert NOT_ENOUGH_FUNDS();

013-H
similar to 012-H, but different location

---------

403, DonationVotingMerkleDistributionBaseStrategy _allocte access control