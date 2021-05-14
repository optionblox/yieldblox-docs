# Governance
The YieldBlox protocol is updated and maintained using a governance token model. Users receive YBX tokens for lending or borrowing. YBX tokens are used to vote on protocol changes and updates using the YieldBlox Governance Turing Signing Server txFunction.

## Distribution
YBX tokens are distributed weekly to users for lending or borrowing with the YieldBlox protocol. Only lends and borrows that originated more than one week before the distribution date are eligible for governance distribution. Governance tokens are distributed using a Governance issuance equation that calculates the total number of governance tokens to distribute based on how many have already been distributed. After the equation calculates how many tokens to distribute, the total distribution is split between the assets supported by the pool based on each asset's governance allocation percentage. The governance allocation percentage is a value set in the data entries of the YBX pool. These values can be modified with the Governance txFunction. Once an asset is assigned the portion of governance tokens to distribute to its lenders, governance tokens are distributed to lenders proportional to the percent of that asset they contributed to the pool.

### Governance Distribution Equation
Used to calculate the total governance tokens to be issued for a given week. This equation is an exponential K distribution with a peak transition to the origin. The number of governance tokens decreases as the total outstanding increases and adjusts to any additional issuance or burning (additional issuance can only occur through a governance proposal).

![\Large](https://latex.codecogs.com/svg.latex?A_%7Bi+1%7D%3Dk%5E%7BB_i&plus;25,000,000%7D)

*A<sub>i+1</sub>*= The issuance of the current week\
*B<sub>i</sub>*= The remaining number of governance tokens to be issued\
*k*=A constant of fit proportions equal to 1.0000003443085\
*A<sub>0</sub>*= The original governance issuance of 3,000,000\
*B<sub>0-1</sub>*=The total number of tokens to be allocated; 15,000,000

#### Governance Issuance Graph

![distribution](https://miro.medium.com/max/700/1*iH3cGw4nU3MPdDuTyRhv_g.png "YBX Distribution")

#### Initial Governance Issuance Distribution
The governance tokens issued initially (the 3,000,000 mentioned in our equation) will be distributed to Script3, the early contributors to our protocol, and our early community members. This initial issuance will be quickly diluted by the governance tokens distributed through the protocol. Below are some charts showing the initial distribution allocation and how it's diluted over time.

![allocation](https://miro.medium.com/max/2587/1*Vq-_E_PFIW1pW1b3Da5_Xw.png "YBX Allocation")


## Governance Proposals
Governance proposals can be created by any account holding more than 1% of outstanding YBX tokens. Creating a proposal generates a governance transaction and a proposal account controlled by the governance contract. The proposal account stores the proposed update’s transaction hash as an account signer and stores the proposal status (either voting or approved) in an account data entry.

## Voting
Users have three days to vote on proposals. They vote by adding a trustline for the asset `YES:proposalAccount` or `NO:proposalAccount` depending on whether they’re voting yes or no for the proposal. At the end of the voting period, votes are tallied by aggregating all accounts with YES or NO trustlines and recording the number of YBX tokens held by each account. If the vote passes (60% of votes are YES), the proposal status data entry is changed to “approved”. If it does not pass, the proposal account is deleted. A proposal vote must reach a quorum of 5% of outstanding governance tokens to pass.

## Proposal Implementation
There is a 2-day delay before implementation after a proposal passes. Proposals are implemented by using the Governance txFunction to sign the update’s transaction hash (which is a proposal account signer) and deleting the proposal account.


