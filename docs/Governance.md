# Governance
The YieldBlox protocol is updated and maintained using a governance token model. Users receive YBX tokens for lending or borrowing. YBX tokens are used to vote on protocol changes and updates using the YieldBlox Governance Turing Signing Server txFunction.

## Distribution
YBX tokens are distributed weekly to users for lending or borrowing with the YieldBlox protocol. Only lends and borrows that originated more than one week before the distribution date are eligible for governance distribution.

## Governance Proposals
Governance proposals can be created by any account holding more than 1% of outstanding YBX tokens. Creating a proposal generates a governance transaction and a proposal account controlled by the governance contract. The proposal account stores the proposed update’s transaction hash as an account signer and stores the proposal status (either voting or approved) in an account data entry.

## Voting
Users have three days to vote on proposals. They vote by adding a trustline for the asset `YES:proposalAccount` or `NO:proposalAccount` depending on whether they’re voting yes or no for the proposal. At the end of the voting period, votes are tallied by aggregating all accounts with YES or NO trustlines and recording the number of YBX tokens held by each account. If the vote passes (60% of votes are YES), the proposal status data entry is changed to “approved”. If it does not pass, the proposal account is deleted. A proposal vote must reach a quorum of 5% of outstanding governance tokens to pass.

## Proposal Implementation
There is a 2-day delay before implementation after a proposal passes. Proposals are implemented by using the Governance txFunction to sign the update’s transaction hash (which is a proposal account signer) and deleting the proposal account.


