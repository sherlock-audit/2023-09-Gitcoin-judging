
002-H
setPoolActive
130.md, 246.md, 378.md provides a valid attack vector. The conditions is that the pool needs to hold overflowed fundings.

002-M
fee on transfer

001-H
High. Unexpected fund loss. front run allocate with register


003-H
125.md
High. Fund loss because of infinite vote.

004-M
useRegistryAnchor
Medium. Core function bricks with high possibility. No fund loss.

005-H
614,260 "also identified 003-H, should submit separatly"
High. voiceCreditsCastToRecipient Accounting error.


005-M
zksync CREAT3 and clone

006-M
recipient.proposalBid > poolAmount revert


007-H
create3 anchor register address



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
last vote determine the status

020-M
re-register resets states to pending

021-M
funding during distribution

024-M
allowedTokens address(0)

025-M
RFP Committee register check

026-M
registration and allocation time overlap
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




049-M
zksync CREATE3

050-M
zksync clone

051-M
DonationVotingMerkleDistribution bitmap corruption
medium, no fund loss

---------

403, DonationVotingMerkleDistributionBaseStrategy _allocte access control



--------- invalids

001
low, bypassing of fee is an acceptable risk

002
invalid, very close to #150, but there is logic error in the submition and the fix is misleading

003
invalid, Allo.sol is not meant to hold funds

004
low, no fund lose, only incompatible with NATIVE token, should consider to fix

005 ^
"low, pool manager's fault to _distribute rejected milestone, should consider to fix"

006
ai generated

007 ^
low, rebase token

008 ^
excess eth refund

010
merkletree alloc for others

011
low, no withdraw in strategies

012
invalid, front run initializer

013
anchor payable execute

014 ^
remove allocator

015
low, directly fund pool lost token, should fix

018 * 
upgrade gap

019 ^
double spending eth

020 ^
metadata not stored

021 ^
low

023 *
invalid, according to sherlock\'s rule

024 *^
invalid, QVBaseStrategy are designed to distribute once, bring sponsor to check

025 *^
merkle root empty
manager's fault, better to fix

026 *^
low/info, rare case, zero transfer revert token

029 *^
invalid, revert in __BaseStrategy_init


#510, #308, #468, #706, #170, #752, #076, #723, #755, #106, #005