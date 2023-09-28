001-L

Valid low, should be fixed. DoS could be avoided by sending extra 1wei of NATIVE.
#074, #101 provides demonstration of fund locking because of this and no refund,
but still low because only 1wei will be lost each call if to workaround this.

002-L

Informational. User' fault if he sends excessive token.

002-H
setPoolActive
130.md, 246.md, 378.md provides a valid attack vector. The conditions is that the pool needs to hold overflowed fundings.

002-M
fee on transfer

001-H
High. Unexpected fund loss. front run allocate with register

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

014-M
reviewRecipients infinit vote

016-M
AccessControlUpgradeable

017-M
Erc20 zero transfer revert

018-M
submitUpcomingMilestone access control

019-M
poolAmount updates qv

020-M
qv reviewing issue

021-M
funding during distribution

022-M
qv reviewing issue status

024-M
allowedTokens address(0)

025-M
RFP Committee register check

027-M
qv time overlap

028-M
setMilestones multiple times

029-M
registry update

030-M
upcomingMilestone status

031-M
other tokens cannot withdraw

032-M
DonationVotingMerkleDistributionBaseStrategy states

033-M
reviewRecipients state

034-M
QvBaseStrategy
_qv_allocate status

035-M
updatePoolTimestamps when active

037-M
stuck fundPool after alloc

038-M
front run reviewRecipients with registerRecipient, medium because conditions

041-M
vote reset milestone RFPCommitteeStrategy

042-M
time overlap

047-M
left

---------

403, DonationVotingMerkleDistributionBaseStrategy _allocte access control



--------- invalids

001
low, bypassing of fee is an acceptable risk

002
invalid, very close to 377, but there is logic error in the submition and the fix is misleading

003
invalid, Allo.sol is not meant to hold funds

004
low, no fund lose, only incompatible with NATIVE token, should consider to fix

005
"low, pool manager's fault to _distribute rejected milestone, should consider to fix"

006
"invalid, using ai generated report will lower your issue ratio and you will be ineligible for payouts"

007
low, rebase token

008
excess eth refund

009
pay fee from allo
allo is not meant to hold funds

010
merkletree alloc for others

011
low, no withdraw in strategies

012
invalid, front run initializer

013
anchor payable execute

014
remove allocator

015
low, directly fund pool lost token, should fix

016
invalid, DonatingVotingMerkleDistributionBaseStrategy is abstract


017
no withdraw native eth

018
upgrade gap

019
double spending eth

020
metadata not stored