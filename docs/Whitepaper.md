# YieldBlox Technical Whitepaper
## A Decentralized Lending Solution Built on Stellar

Markus Paulson-Luna: markus@script3.io\
Andrew Pierskalla: andrew@optionblox.io\
Alexander Mootz: alex@optionblox.io

<p>&nbsp;</p>

## Abstract

This whitepaper covers the technical details of YieldBlox’s decentralized lending protocols. These details include explanations of the protocol tokens, accounts, and TSS(Turing Signing Server) txFunctions (Smart Contracts) that encompass the YieldBlox lending protocol. This whitepaper will be continually updated as details regarding YieldBlox’s core protocols change. Currently it covers the base protocol structure and math.

<p>&nbsp;</p>

## Table of Contents:
- [Introduction](#introduction)
- [Protocol Structure](#protocol-structure)
- [Lending Protocol txFunctions](#lending-txfunctions)
- [Protocol Math](#math)

<p>&nbsp;</p>

## Introduction:
The YieldBlox protocol serves as a tier-3 blockchain app, a layer between YieldBlox web-app users and the Stellar ledger. Users link their wallets to YieldBlox’s web-app, which takes user inputs and communicates them to the YieldBlox protocol. The protocol uses TSS(Turing Signing Server) txFunctions to build and sign Stellar transactions that carry out protocol operations. Users approve these transactions with their wallet and receive the result of their transaction, whether that’s derivative tokens, collateral deposits, or underlying assets.

### High-Level Protocol Diagram

 ![alt text](https://raw.githubusercontent.com/optionblox/yieldblox-docs/main/media/High-Level%20YBX%20protocol.png "YBX Protocol")

## Protocol Structure
The YieldBlox protocol facilitates lending by utilizing a network of protocol accounts, tokens, and claimable balances.
### Loan Terms
Loans can be taken for any time period as long as the user continues to maintain a collateral balance greater than 110% of the value of their loan plus their loan’s accrued interest. If the collateral balance falls below 110% of the loan value plus the value of the loan’s accrued interest, the loan can be liquidated by another protocol participant.

### Protocol Data Feeds
The YieldBlox protocol relys on liability, utilization, and price feeds to function. 
#### Liability Feed
The protocol tracks it's total liabilities by sending liability tokens to the YieldBlox Feed account whenever a loan is taken out and by burning those tokens whenever it is repaid. This is necessary to allow the protocol and protocol data consumers to quickly calculate total pool liabilities.
### Utilization Feed
The protocol tracks it's utilization over time by sending utilization tokens to the YieldBlox Feed account whenever a loan is taken out, pool tokens are minted, or pool tokens are burned. This is necessary to allow the protocol to determine the average utilization rate over the period of a loan, which is required to calculate a loans accrued interest fees.
### Price Feed
The protocol tracks the value of user's collateral with a price feed that pulls price data from centralized exchanges like Coinbase, averages them, and sanity checks them against the DEX TWAP(time-weighted-average-price). This is only a temporary solution as a dedicated oracle solution for TSS apps is currently in the works. More details on this protocol are available on our docs page: https://docs.yieldblox.com/#/

### Protocol Tokens
#### Pool Tokens
YieldBlox pool tokens are sent to protocol lenders in exchange for the assets they lend the protocol. These tokens represent the proportion of the associated underlying asset in the pool that the user owns. The value of pool tokens increases over time as the pool intakes interest fees. There is a separate pool token for each asset held in the YieldBlox lending pool.\
**Token Identification:**\
Pool tokens are issued by the YieldBlox lending pool account
The naming convention for pool token’s code is `y[underlying-asset-code]`
#### Liability Tokens
Liability tokens are used to track users' borrowed assets. They are created and sent to the tracker account when a borrower takes out a loan and burned when that loan is repaid or liquidated. In addition, the pool creates a claimable balance of liability tokens when a loan is taken out that is claimable by the borrower in 100 years. This serves to track a borrowers liabilities and the time they have been outstanding. The claimable balance is deleted when the loan is repayed. There is a separate liability token for each asset held in the YieldBlox lending pool.\
**Token Identification:**\
Liability tokens are issued by the YieldBlox lending pool account
The naming convention for liability token’s code is `l[underlying-asset-code]`
#### Utilization Tokens
Utilization tokens are used to track the pool's utilization ratios (the percent of the pool that is lent out) for it’s asset balances. Whenever the borrow txFunction is ran the contract sends a utilization token payment to the utilization tracker account. The payment amount reflects the loan pool’s current utilization ratio for the asset being lent out. There is a separate utilization token for each asset held in the YieldBlox lending pool.\
**Token Identification:**\
Utilization tokens are issued by the YieldBlox lending pool account
The naming convention for the utilization token’s code is `u[underlying-asset-code]`
#### Governance Token
YieldBlox uses a governance token based governance model. Users receive governance tokens for participating in the YieldBlox protocol by lending or borrowing. They can then use these tokens to create governance proposals to modify the YieldBlox protocol and to vote on governance proposals. For information on governance token tokenomics see our docs page: https://docs.yieldblox.com/#/\
**Token Identification:**\
Governance tokens are issued by the YieldBlox lending pool account
The asset code for governance tokens is `YBX`
### Protocol Accounts
#### YieldBlox Lending Pool
The YieldBlox Lending pool holds all assets deposited by lenders and lends them out to borrowers. It also issues pool tokens to lenders and governance tokens to lenders and borrowers. Finally, it stores critical protocol information in its data entries and claimable balances.

**Account Claiamable Balances**\
1. *Liability Claimable Balances*: Used to track a users loan. When a user takes out a loan the lending pool will create a claimable balance of liability tokens equal to the amount of underlying tokens lent to the user. The pool will be able to claim this claimable balance at any theim and the user will be able to claim this claimable balance in 100 years. This allows the protocol to easily discover a users liabilities and the amount of time that the liabilities have been outstanding. The claimable balance is deleted when the user repays the loan or when the loan is liquidated. 

**Account Data Entries**
1. *Interest rate numerator*: Used in the interest rate calculation. Seperate data entry for every pool asset type.
2. *Utilization factor*: Used in the interest rate calculation. Seperate data entry for every pool asset type.
3. *Utilization addend*: Used in the interest rate calculation. Seperate data entry for every pool asset type.
4. *Approved Collateral*: List of approved collateral assets for an underlying asset. There is a separate data entry for each approved collateral asset and a separate list for each supported underlying asset. 
5. *Base Collateral Factor*: Maximum Loan to Value ratio for an underlying asset. There is a seperate data entry for each approved collateral asset. 
6. *Liquidation Level*: Collateral Value to (Loan Value + Accrued Interest) ratio where a loan can be liquidated. 
7. *Liquidation penalty*: User penalty for having a loan liquidated. Paid to the loan liquidator (the user who runs the liquidation txFunction). 
8. *Liquidate Signers*: TSS signers for the Liquidate txFunction. There is a separate data entry for every signer.
9. *Repay Signers*: TSS signers for the Repay txFunction. There is a separate data entry for every signer.

**Trustlines**
- All supported underlying assets
- All supported collateral assets

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
This account receives utilization token payments and liability token payments from the YieldBlox lending pool. These payments are aggregated to calculate the average utilization ratio for a loan and the pools total utilization.

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
While the User Account is not a protocol account in the sense that it is controlled by the user, it still plays a critical role in the protocol by holding the users collateral balance in a claimable balance.

**Claimable Balances**
- *Collateral Claimable Balance*: This claimable balance is used to provide collateral to the protocol when the user takes out a loan. If the user wishes to deposit additional collateral they can create additional claimable balances in their account. The claimable balances must be structed to be claimable by the pool at any time and claimable by the user in 100 years. When a user repays their loan or it is liquidated the pool will claim the claimable balance and either return it to the user (if the loan was properly repayed), or confiscate it and pay it to the loan liquidator (if the loan was liquidated).

### Lending txFunctions
This section provides a brief overview of the txFunctions involved in the YieldBlox lending protocol. Our txFunctions will be publicly available on the YieldBlox GitHub repository once we officially launch our beta. If you would like more technical details on the txFunctions feel free to contact us.
1. *Mint txFunction*\
Used to lend assets to the pool in exchange for pool tokens and to borrow from the pool by providing collateral. To lend assets, users deposit the assets they wish to lend in the lending pool and are sent pool tokens proportional to the percentage of the pool's balance of the asset that the user provided. To borrow assets the user creates a claimable balance of pool tokens that act as collateral for the loan they wish to take out and are sent their borrowed assets. If they provide normal assets as collateral, the assets will be converted into pool tokens before being locked in the claimable balance. Since pool tokens are used as collateral the user will generate interest on their collateral deposit.\
    - Calculations Used:
        1. Minimum Collateral Requirement
        2. Pool Tokens Issued
2. *Burn txFunction*\
Used to convert pool tokens into their underlying assets. Users burn the pool tokens by sending them to the lending pool and are sent the assets associated with the pool tokens.
    - Calculations Used:
        1. Asset payout
3. *Repay txFunction*\
Used to repay loans. Users send the repayment to the pool and are returned their collateral balance minus all accrued interest fees.
    - Calculations Used:
        1. Interest Accrued
4. *Liquidate txFunction*\
Used to liquidate a delinquent loan. Loans can be liquidated when they reach a 97% (loan value)+(interest accrued) to (collateral value) ratio. The liquidator repays the delinquent loan and is send the value of their repayment in collateral along with a small liquidation incentive. The rest of the collateral balance is sent to the pool.
    - Calculations used:
        1. Asset Payout: Used to value the pool tokens being used as collateral
        2. Interest Accrued
        3. Liquidation Penalty
5. *Governance txFunction*\
Used to modify the protocol in any way from tweaking the interest rate calculation to modifying the core txFunctions. This is a three stage txFunction. 
    1. First Stage: The first stage creates a governance proposal. This creates a proposal account that contains the proposed transaction hash as a signer. Users vote on the proposal by adding trustlines for `YES:proposalAccount` or `NO:proposalAccount assets`. 
    2. Second Stage: At the end of a 3 day period the votes are tallied by running the governance txFunction again and providing the public key of the proposal account. This is the second stage of the txFunction. Each account's vote is worth the number of governance tokens that the account holds. If a proposal passes then the proposal account marks itself as approved using an account data entry. 
    3. Third Stage: After 2 days have passed the governance txFunction is ran a third time and provided the proposal accounts public key to carry out the third stage of the txFunction. This stage signs the proposed hash approving it to be submitted.  
6. *Flash txFunction*\
This txFunction allows users to take out flash loans. Users can borrow any amount of assets from the lending pool as long as they repay the assets in the same transaction envelope that they borrow them in. This txFunction will not be turned on initially.
### Protocol Calculations
#### Utilization Ratio Calculations
Used to calculate the current utilization ratio for an asset

![\Large](https://latex.codecogs.com/svg.latex?U%3D1-%5Cfrac%7BL%7D%7BL&plus;B%7D)

*U* = Current Utilization Ratio\
*L*= Total Liability Tokens. Tracked by YBX Tracker\
*B*=Total Pool Balance

##### Aggregated Utilization Ratio
Used to calculate the aggregated utilization ratio for a loan. 

![\Large](https://latex.codecogs.com/svg.latex?U_a%20%3D%20%5Csum_%7Bi%3DT_b%7D%5E%7BT_r%7D%5Cfrac%7BA_i*%28T_i-T_%7Bi-1%7D%29%7D%7BT_i-T_%7Bi-1%7D%7D)

*T<sub>r</sub>*= Time the loan was repaid at\
*T<sub>b</sub>*=Time the loan was borrowed at\
*A*=Amount of the payment to the utilization tracker. Equal to the utilization ratio at the time of payment\
*T*=Time of the payment in milliseconds

#### Interest Rate Calculations
YieldBlox uses a purely demand based interest rate calculation, which means the only protocol variable involved in the equation is the utilization rate. This allows the protocol to more efficiently adjust for market conditions without having to change a base interest rate. It also gives governance token holders a large amount of flexibility when protocol interest rates. The equation involves the Interest Numerator, Utilization Addend, and Utilization Factor constants which can be updated to change not only the slope of the interest rate curve but also the curve's exponentiality and y-intercept. Currently these constants are set at 10, 1.6, and -0.45 respectivley which results in the following curve.

 ![alt text](https://github.com/optionblox/yieldblox-docs/raw/main/media/YBX%20Interest%20Graph.png "Interest Rate Curve")

##### Interest Rate
Used to calculate the current interest rate.

![\Large](https://latex.codecogs.com/svg.latex?I%20%3D%28%5Cfrac%7Ba%7D%7B%281&plus;%2810e%29%5E%7B%28b&plus;c*U_r%29%7D%29%7D%29)

*I*= Interest Rate\
*U<sub>r</sub>*= Utilization Ratio\
*a*= Interest Numerator. Set by pool\
*b*= Utilization Addend. Set by pool\
*c*= Utilization Factor. Set by pool

##### Interest Accrued
Used to calculate the interest fee accrued by a loan.

![\Large](https://latex.codecogs.com/svg.latex?I_a%20%3DA*%28%5Cfrac%7Ba%7D%7B%281&plus;%2810e%29%5E%7B%28b&plus;c*U_a%29%7D%29%7D%29)

*I<sub>a</sub>*= Accrued interest\
*A*= Loan amount\
*U<sub>a</sub>*= Aggregated Utilization Ratio\
*a*= Interest Numerator. Set by pool\
*b*= Utilization Addend. Set by pool\
*c*= Utilization Factor. Set by pool

#### Min Collateral Requirement
Used to calculate the Minimum Collateral Requirement for a loan.

![\Large](https://latex.codecogs.com/svg.latex?C_m%20%3D%20%5Cfrac%7BA%7D%7BC_b%7D)

*C<sub>m</sub>*=  Minimum Collateral Requirement\
*A*= Loan Amount\
*C<sub>b</sub>*= Base Collateral Factor. Set by pool

#### Pool Tokens Issued
Used to calculate the number of pool tokens issued to a lender.

![\Large](https://latex.codecogs.com/svg.latex?P_i%20%3D%20%5Cfrac%7B%5Cfrac%7BR_n%7D%7BR_c&plus;L%7D*P_o%7D%7B1-%5Cfrac%7BR_n%7D%7BR_c&plus;L%7D%7D)

*P<sub>i</sub>*= New pool tokens that will be issued to the lender\
*P<sub>o</sub>*= Current outstanding pool tokens\
*R<sub>n</sub>*= Assets the user is adding to the lending pool\
*R<sub>c</sub>*= Current pool reserves of the asset the user is lending\
*L*= Current outstanding liability tokens for the asset the user is lending

#### Asset payout
Used to calculate the asset payout for a given number of burned pool tokens.

![\Large](https://latex.codecogs.com/svg.latex?P%20%3D%20%28A_b&plus;L%29%5Cfrac%7BP_b%7D%7BP_t%7D)

*P*= Amount of the asset payed out\
*A<sub>b</sub>*= Pool balance of the underlying asset of the pool token being burned\
*L*= Total liability tokens outstanding for the underlying asset of the pool token being burned\
*P<sub>b</sub>*= Number of pool tokens burned\
*P<sub>t</sub>*= Total number of pool tokens




