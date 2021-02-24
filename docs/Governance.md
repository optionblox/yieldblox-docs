# Governance
The YieldBlox protocol is updated and maintained using a governance token model. Users receive YBX tokens for lending or borrowing. YBX tokens can be used to vote on protocol changes and updates using the YieldBlox Governance Turing Signing Server txFunction.

## Distribution
YBX tokens are distributed to users when the derivatives they have written expire or are exercised. The amount of governance tokens that are distributed depends on the value of options that were written and the amount of time they were outstanding. In the initial version of the protocol YBX token distribution will be limited to options for certain asset pairings.

## Governance Proposals
Governance proposals can be created by any account holding more than .1% of outstanding YBX tokens. Creating a proposal generates a governance transaction and a proposal account controlled by the governance contract. The proposal account stores the proposed governance transaction hash as an account signer and stores the proposal status (either voting, approved, or denied) in an account data entry.

## Voting
Users have three days to vote on proposals. They vote by adding a trustline for the asset `YES:proposalAccount` or `NO:proposalAccount` depending on whether they’re voting yes or no for the proposal. At the end of the voting period votes are tallied by getting all accounts with YES or NO trustlines and recording the number of YBX tokens held by each account. If the vote passes (60% of votes are YES) the proposal status data entry will be changed to approved, if it does not pass the entry will be changed to denied. A proposal vote must reach a quorum of 5% of outstanding governance tokens in order to pass.

## Proposal Implementation
After a proposal passes there is a 2 day delay, then proposals are implemented by using the Governance txFunction to sign the transaction hash in the proposal account data entry and delete the proposal account.


