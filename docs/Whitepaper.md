
# YieldBlox Technical Whitepaper
## A Decentralized Lending Solution Built on Stellar

Markus Paulson-Luna: markus@script3.io\
Andrew Pierskalla: andrew@script3.io\
Alexander Mootz: alex@script3.io

<p>&nbsp;</p>

## Abstract

This whitepaper covers the technical details of YieldBlox's decentralized lending protocol. These details include explanations of the protocol tokens, accounts, and TSS(Turing Signing Server) txFunctions(Smart Contracts) that encompass the YieldBlox lending protocol. This whitepaper will be continually updated as details regarding YieldBlox's protocol change. Currently, it covers the base protocol structure and math.

<p>&nbsp;</p>

## Table of Contents:
- [Introduction](#Introduction)
- [Protocol Structure](#protocol-structure)
- [Lending Protocol txFunctions](#lending-txfunctions)
- [Protocol Math](#protocol-calculations)

<p>&nbsp;</p>

## Introduction:
The YieldBlox protocol serves as a tier-3 blockchain app, a layer between YieldBlox web-app users and the Stellar ledger. Users link their wallets to YieldBlox's web-app, which takes user inputs and communicates them to the YieldBlox protocol. The protocol uses TSS txFunctions to build and sign Stellar transactions that carry out protocol operations. Users approve these transactions with their wallet and receive the result of their transaction, whether that's pool tokens or a loan.

### High-Level Protocol Diagram

 ![protocol](_media/ybxProtocolHighLevel.png "YBX Protocol")

## Protocol Structure
The YieldBlox protocol facilitates lending by utilizing a network of protocol accounts, tokens, and claimable balances.
### Loan Terms
Loans can be taken out for any time period as long as the user maintains a collateral balance greater than 110% of the value of their loan plus their loan's accrued interest. If the collateral balance falls below 110% of the loan value plus the value of the loan's accrued interest, the loan can be liquidated by another protocol participant.

### Protocol Data Feeds
The YieldBlox protocol relies on liability, utilization, and price feeds to function. 
#### Liability Feed
The protocol tracks its total liabilities by sending liability tokens to the YieldBlox Tracker account when a loan is originated and by burning those tokens when it is repaid. This process is necessary to allow the protocol and protocol data consumers to calculate total pool liabilities quickly.
### Utilization Feed
The protocol tracks its utilization over time by sending utilization tokens to the YieldBlox Tracker account whenever a loan is originated, pool tokens are minted, or pool tokens are burned. This process is necessary to allow the protocol to determine the average utilization rate over a loan's lifecycle, which is required to calculate a loan's accrued interest fees.
### Price Feed
The protocol tracks users' collateral value with a price feed that pulls price data from centralized exchanges like Coinbase, averages them, and sanity checks them against the DEX TWAP(time-weighted-average-price). The price feed is only a temporary solution as a dedicated oracle solution for TSS apps is currently in the works. More details on this feed are available on our docs page: https://docs.yieldblox.com/#/

### Protocol Tokens
#### Pool Tokens
YieldBlox pool tokens are sent to protocol lenders in exchange for the assets they lend the protocol. These tokens represent the portion of the associated underlying asset in the pool that the lender owns. The value of pool tokens increases over time as the pool intakes interest fees. There is a separate pool token for each asset held in the YieldBlox lending pool.

**Token Identification:**
- Pool tokens are issued by the YieldBlox lending pool account
- The naming convention for pool token's code is `y[underlying-asset-code]`
#### Liability Tokens
Liability tokens track users' borrowed assets. They are created and sent to the YieldBlox Tracker account when a borrower originates a loan and burned when that loan is repaid or liquidated. In addition, the pool creates a claimable balance of liability tokens when a loan is originated that is claimable by the borrower in 100 years. This claimable balance tracks a borrower's liabilities and the time they have been outstanding. When the borrower repays their loan, the claimable balance is deleted. There is a separate liability token for each asset held in the YieldBlox lending pool.

**Token Identification:**
- Liability tokens are issued by the YieldBlox lending pool account
- The naming convention for liability token's code is `l[underlying-asset-code]`
#### Utilization Tokens
Utilization tokens track the pool's asset utilization ratios(the percentage of the pool's balance of that asset currently lent out). Whenever a txFunction modifies the pool's asset balances, the txFunction sends a utilization token payment to the YieldBlox Tracker account. The payment amount reflects the loan pool's current utilization ratio for the asset that had its balance modified. There is a separate utilization token for each asset held in the YieldBlox lending pool.

**Token Identification:**
- Utilization tokens are issued by the YieldBlox lending pool account
- The naming convention for the utilization token's code is `u[underlying-asset-code]`
#### Governance Token
YieldBlox uses a token-based governance model. Users receive governance tokens for participating in the YieldBlox protocol by lending or borrowing. They can then use these tokens to create governance proposals that modify the YieldBlox protocol and vote on governance proposals. For information on the YileBlox governance system, see our docs page: https://docs.yieldblox.com/#/

**Token Identification:**
- Governance tokens are issued by the YieldBlox lending pool account
- The asset code for governance tokens is `YBX`
### Protocol Accounts
#### YieldBlox Lending Pool
The YieldBlox Lending Pool holds all assets deposited by lenders and lends them out to borrowers. It also issues pool tokens to lenders and governance tokens to lenders and borrowers. Finally, it stores critical protocol information in its data entries and claimable balances.

**Account Claiamable Balances**
1. *Liability Claimable Balances*: Used to track a user's loan value. When a user originates a loan, the lending pool creates a claimable balance of liability tokens equal to the number of tokens borrowed. The pool can claim this claimable balance at any time, and the user can claim it in 100 years. This claimable balance structure allows the protocol to easily discover a user's liabilities and the amount of time the liabilities have been outstanding. The claimable balance is deleted when the user repays the loan or when the loan is liquidated. 

**Account Data Entries**
1. *Interest rate numerator*: Used in the interest rate calculation. Separate data entry for every asset supported by the pool.
2. *Utilization factor*: Used in the interest rate calculation. Separate data entry for every asset supported by the pool.
3. *Utilization addend*: Used in the interest rate calculation. Separate data entry for every asset supported by the pool.
4. *Approved Collateral*: List of approved collateral assets for an underlying asset. There is a separate data entry for each approved collateral asset and a separate list for each supported asset. 
5. *Base Collateral Factor*: Maximum Loan to Value ratio for an asset. There is a separate data entry for each approved collateral asset. 
6. *Liquidation Level*: Collateral Value to (Loan Value + Accrued Interest) ratio where a loan can be liquidated. Separate data entry for every asset supported by the pool.
7. *Liquidation penalty*: User penalty for having a loan liquidated. This penalty is paid to the loan liquidator (the user who runs the liquidation txFunction). 

**Trustlines**
- All supported assets

**Signature structure**\
*Thresholds*
- Low Threshold: 20
- Medium Threshold:20
- High Threshold:30

*Signers*
- Burn txFunction: 3 signers, weight 10
- Mint txFunction: 3 signers, weight 10
- Repay txFunction: 3 signers, weight 10
- Liquidate txFunction: 3 signers, weight 10
- Flash txFunction: 3 signers, weight 10
- Governance txFunction: 5 signers, weight 10
#### YieldBlox Tracker
This account receives utilization token payments and liability token payments from the YieldBlox lending pool. These payments are aggregated to calculate the average utilization ratio for a loan and the pool's total utilization.

**Trustlines**
- Utilization token
- Liability token

**Signature structure**\
*Thresholds*
- Low Threshold: 20
- Medium Threshold:20
- High Threshold:30

*Signers*
- Repay txFunction: 3 signers, weight 10
- Liquidate txFunction: 3 signers, weight 10
- Governance txFunction: 5 signers, weight 10
#### User Account
While the User Account is not a protocol account in the sense that the protocol controls it, it still plays a critical role by holding the user's collateral balance in a claimable balance.

**Claimable Balances**
- *Collateral Claimable Balance*: The Mint txFunction creates this claimable balance to provide collateral to the protocol when the user takes out a loan. If the user wishes to deposit additional collateral, they can create additional claimable balances in their account. The claimable balances must be structured to be claimable by the pool at any time and claimable by the user in 100 years. When a user's loan is repaid or liquidated, the pool claims the claimable balance and either returns it to the user (if the loan was properly repayed), or confiscates it and pays it to the loan liquidator (if the loan was liquidated).

### Lending txFunctions
This section provides a brief overview of the txFunctions involved in the YieldBlox lending protocol. The full txFunctions will be publicly available on the YieldBlox GitHub repository once we officially launch our beta. If you would like more technical details on the txFunctions, feel free to contact us.
1. *Mint txFunction*\
The Mint txFunction is used to both lend assets to the pool in exchange for pool tokens and borrow from the pool by providing collateral. To lend assets, users deposit the assets they wish to lend in the lending pool and receive pool tokens proportional to the percentage of the pool's balance of the deposited asset. To borrow assets, the user creates a claimable balance of pool tokens that act as collateral for the loan they wish to take out and receive their borrowed assets. If they provide normal assets as collateral, the Mint txFunction converts the assets into pool tokens before locking them in a claimable balance. Since pool tokens are used as collateral, the user receives interest on their collateral deposit.\
    - Calculations Used:
        1. Minimum Collateral Requirement
        2. Pool Tokens Issued
2. *Burn txFunction*\
The Burn txFunction is used to convert pool tokens into underlying assets. Users burn their pool tokens and receive the assets associated with the pool tokens.
    - Calculations Used:
        1. Asset payout
3. *Repay txFunction*\
The Repay txFunction is used to repay loans. Users send the repayment to the pool and receive their collateral balance net of accrued interest fees.
    - Calculations Used:
        1. Interest Accrued
4. *Liquidate txFunction*\
The Liquidate txFunction is used to liquidate a delinquent loan. Loans can be liquidated when they reach a 95% (loan value)+(interest accrued) to (collateral value) ratio. The liquidator repays the delinquent loan and is send the value of their repayment in collateral along with a small liquidation incentive. The rest of the collateral balance is sent to the pool.
    - Calculations used:
        1. Asset Payout: Used to value the pool tokens used collateral
        2. Interest Accrued
        3. Liquidation Penalty
5. *Governance txFunction*\
The Governance txFunction is used to modify the protocol, from tweaking the interest rate calculation to modifying the core txFunctions. Governance is a three-stage txFunction. 
    1. First Stage: The first stage creates a governance proposal by creating a proposal account that contains the proposed update's transaction hash as a signer. Users vote on the proposal by adding trustlines for `YES:proposalAccount` or `NO:proposalAccount assets`. 
    2. Second Stage: At the end of a 3 day period, the votes are tallied by running the Governance txFunction again and providing the proposal account's public key. Each user's vote is worth the number of governance tokens that their account holds. If a proposal passes, then the proposal account marks itself as approved using an account data entry. If it fails the proposal account is deleted.
    3. Third Stage: After 2 days have passed, the governance txFunction is ran a third time and provided the proposal account's public key to carry out the third stage of the txFunction. This stage signs the proposed update's transaction hash approving it to be submitted.  
6. *Flash txFunction*\
This txFunction allows users to take out flash loans. Users can borrow any amount of assets from the lending pool as long as they repay the assets in the same transaction envelope that they borrow them in. This txFunction will not be turned on initially.
### Protocol Calculations
#### Utilization Ratio Calculations
Used to calculate the current utilization ratio for an asset

![\Large](https://latex.codecogs.com/svg.latex?U%3D1-%5Cfrac%7BL%7D%7BL&plus;B%7D)

*U* = Current Utilization Ratio\
*L* = Total Liability Tokens. Tracked by YBX Tracker\
*B* = Total Pool Balance

##### Aggregated Utilization Ratio
Used to calculate the aggregated utilization ratio for a loan. 

![\Large](https://latex.codecogs.com/svg.latex?U_a%20%3D%20%5Csum_%7Bi%3DT_b%7D%5E%7BT_r%7D%5Cfrac%7BA_i*%28T_i-T_%7Bi-1%7D%29%7D%7BT_i-T_%7Bi-1%7D%7D)

*T<sub>r</sub>* = Time the loan was repaid at\
*T<sub>b</sub>* = Time the loan was borrowed at\
*A* = Amount of the payment to the utilization tracker. Equal to the utilization ratio at the time of payment\
*T* = Time of the payment in milliseconds

#### Interest Rate Calculations
YieldBlox uses a purely demand-based interest rate calculation, which means the only protocol variable involved in the equation is the utilization rate. This equation allows the protocol to more efficiently adjust for market conditions without changing a base interest rate. It also gives governance participants a large amount of flexibility when modifying protocol interest rates. The equation involves the Interest Numerator, Utilization Addend, and Utilization Factor constants which can be updated to change not only the slope of the interest rate curve, but also the curve's exponentiality and y-intercept. These constants are currently 10, 1.6, and -0.45, respectively, which results in the following curve.

 ![apr](_media/ybxInterestRates.png "Interest Rate Curve")

##### Interest Rate
Used to calculate the current interest rate.

![\Large](https://latex.codecogs.com/svg.latex?I%20%3D%28%5Cfrac%7Ba%7D%7B%281&plus;%2810e%29%5E%7B%28b&plus;c*U_r%29%7D%29%7D%29)

*I* = Interest Rate\
*U<sub>r</sub>* = Utilization Ratio\
*a* = Interest Numerator. Set by pool\
*b* = Utilization Addend. Set by pool\
*c* = Utilization Factor. Set by pool

##### Interest Accrued
Used to calculate the interest fee accrued by a loan.

![\Large](https://latex.codecogs.com/svg.latex?I_a%20%3DA*%28%5Cfrac%7Ba%7D%7B%281&plus;%2810e%29%5E%7B%28b&plus;c*U_a%29%7D%29%7D%29)

*I<sub>a</sub>* = Accrued interest\
*A* = Loan amount\
*U<sub>a</sub>* = Aggregated Utilization Ratio\
*a* = Interest Numerator. Set by pool\
*b* = Utilization Addend. Set by pool\
*c* = Utilization Factor. Set by pool

#### Min Collateral Requirement
Used to calculate the Minimum Collateral Requirement for a loan.

![\Large](https://latex.codecogs.com/svg.latex?C_m%20%3D%20%5Cfrac%7BA%7D%7BC_b%7D)

*C<sub>m</sub>* =  Minimum Collateral Requirement\
*A*= Loan Amount\
*C<sub>b</sub>* = Base Collateral Factor. Set by pool

#### Pool Tokens Issued
Used to calculate the number of pool tokens issued to a lender.

![\Large](https://latex.codecogs.com/svg.latex?P_i%20%3D%20%5Cfrac%7B%5Cfrac%7BR_n%7D%7BR_c&plus;L%7D*P_o%7D%7B1-%5Cfrac%7BR_n%7D%7BR_c&plus;L%7D%7D)

*P<sub>i</sub>* = New pool tokens that will be issued to the lender\
*P<sub>o</sub>* = Current outstanding pool tokens\
*R<sub>n</sub>* = Assets the user is adding to the lending pool\
*R<sub>c</sub>* = Current pool reserves of the asset the user is lending\
*L*= Current outstanding liability tokens for the asset the user is lending

#### Asset payout
Used to calculate the asset payout for a given number of burned pool tokens.

![\Large](https://latex.codecogs.com/svg.latex?P%20%3D%20%28A_b&plus;L%29%5Cfrac%7BP_b%7D%7BP_t%7D)

*P*= Amount of the asset paid out\
*A<sub>b</sub>* = Pool balance of the underlying asset of the pool token being burned\
*L* = Total liability tokens outstanding for the underlying asset of the pool token being burned\
*P<sub>b</sub>* = Number of pool tokens burned\
*P<sub>t</sub>* = Total number of pool tokens




