
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
Loans can be taken out for any time period as long as the user maintains an account health factor above 1 (an accounts health factor is based on the accounts total liability value, collateral value, and collateral liquidation factors). If the accounts health factor falls below 1, the accounts loans can be liquidated by another protocol participant until their account health factor increases to 1.01.
### Loan Interest Rates
Users can borrow from YieldBlox at either fixed or floating rates. Floating rates are based purely on demand and will fluctuate based on the borrowed assets utilization ratio. Fixed rates are a multiple of the floating rate at loan origination. They will always be higher than floating rates but some borrowers may prefer the stability of fixed rates rates. A loans fixed rate can be force rebalanced up to the current stable rate if the loans rate has fallen below the market floating rate. This is accomplished with the swapRate txFunction. User's can also use the swapRate txFunction to swap their loan from fixed to the current floating rate or from floating to the current fixed rate.
### Protocol Data Feeds
The YieldBlox protocol relies on utilization and price feeds to function. 
#### Utilization Feed
The protocol tracks its utilization over time with a YieldBlox Tracker account. Whenever a protocol transaction increases an assets utilization ratio the tracker account sends utilization tokens to the pool account. Whenever a protocol transaction decreases an assets utilization ratio the pool account sends utilization tokens back to the tracker account and the tracker account tracks this action in its operations history by sending an identical payment to itself. This process is necessary to allow the protocol to determine the average utilization rate over a loan's lifecycle, which is required to calculate a loan's accrued interest fees. The amount of tokens sent reflects the utilization delta of the payment before last multiplied by the number of blocks between the secont to last and last utilization modifying transaction. 

For example, given the following ledger state:
1. Second to last utilization modifying transaction: Occured on block 1000 and Borrowed 100 XLM increasing the utilization ratio by .1
2. Last utilization modifying transaction: Occured on block 1100, the utilization delta of this transaction doesn't matter
3. Current transaction: Will include a utilization delta payment of 10 utilization tokens (100 blocks * .1 utilization delta)

#### Price Feed
The protocol tracks users' collateral value with a price feed that pulls price data from centralized exchanges like Coinbase, averages them, and sanity checks them against the DEX TWAP(time-weighted-average-price). The price feed is only a temporary solution as a dedicated oracle solution for TSS apps is currently in discussion. More details on this feed are available on our docs page: https://docs.yieldblox.com/#/

### Protocol Tokens
#### Protocol Token Code Components
There are a few special characters we use in the codes of protocol tokens to allow the protocol to identify the underlying asset the protocol token is associated with.
1. Asset overlap codes: YieldBlox must be able to support 12 character asset codes as those are the longest asset codes supported by the Stellar network. However, all protocol token codes have 3 protocol specific characters. This means that we can only include the first 9 characters of an underlying asset code in the protocol asset code. We handle this issue by inluding an asset overlap code as one of the 3 protocol specific characters. If there are multiple underlying assets supported by the protocol that have assets code longer than 9 characters and also share the first 9 characters of their asset codes we use asset overlap codes to differentiate them. For example, say YieldBlox supports an asset with the code `sp500fundGrw` and an asset with the code `sp500fundVal`, since the assets share the first 9 characters (`sp500fund`) the protocol would assign asset overlap code `1` to `sp500fundGrw` and `2` to `sp500fundVal`. `sp500fundGrw`'s protocol token code would be `[protocol_char][Issuer_Code][1][sp500fund]` and `sp500fundVal`'s protocol token code would be `[protocol_char][Issuer_Code][2][sp500fund]`. If a third overlapping asset, say `sp500fund3x`, was added, it's asset overlap code would be `2`. The asset overlap codes are formatted in base64 so the protocol can currently support up to 64 overlapping assets. Asset overlap codes are tied to data entries on the yieldblox lending pool which the protocol can use to find the full underlying asset code. If the underlying asset code is less than 10 characters long the asset_overlap_code will always be `0`
2. Underlying Issuer Code: YieldBlox must be able to support assets with identical codes but different issuing accounts as these assets are supported and relatively common on the Stellar network. We do so by adding a underlying issuer code character to protocol assets. This is a base64 character that identifies the issuing account of the underlying code. For example, say we support an asset with the code `ETH` which is issued by the account `GCNSGHUCG5VMGLT5RIYYZSO7VQULQKAJ62QA33DBC5PPBSO57LFWVV6P` and an asset with the code `ETH` which is issued by the account `GBETHKBL5TCUTQ3JPDIYOZ5RDARTMHMEKIO2QZQ7IOZ4YC5XV3C2IKYU`. The first `ETH` asset will be assigned the underlying issuer code `0` and the second `ETH` asset will be assigned the underlying issuer code `1`. So the first asset's protocol token code will be `[protocol_char][0][asset_overlap_code][ETH]` and the second asset's protocol token code will be `[protocol_char][1][asset_overlap_code][ETH]` Underlying issuer codes are tied to data entries on the yieldblox lending pool so the protocol can find the full issuer account ID. 
#### Pool Tokens
YieldBlox pool tokens are sent to protocol lenders in exchange for the assets they lend the protocol. These tokens represent the portion of the associated underlying asset in the pool that the lender owns. The value of pool tokens increases over time as the pool intakes interest fees. There is a separate pool token for each asset held in the YieldBlox lending pool.

**Token Identification:**
- Pool tokens are issued by the YieldBlox lending pool account
- The naming convention for pool token's code is `y[underlying-issuer-code][asset-overlap-code][underlying-asset-code]`

#### Liability Tokens
Liability tokens track users' borrowed assets. The pool creates a claimable balance of liability tokens when a loan is originated that is claimable by the borrower in 100 years and claimable by the pool at any time. This claimable balance tracks a borrower's liabilities and the time they have been outstanding. When the borrower repays their loan, the claimable balance is deleted. There is a separate liability token for each asset held in the YieldBlox lending pool.

**Token Identification:**
- Liability tokens are issued by the YieldBlox lending pool account
- The naming convention for liability token's code is `l[underlying-issuer-code][asset-overlap-code][underlying-asset-code]`

#### Utilization Tokens
Utilization tokens track the pool's asset utilization ratios(the percentage of the pool's balance of that asset currently lent out). 
There is a separate utilization token for each asset held in the YieldBlox lending pool.

**Token Identification:**
- Utilization tokens are issued by the YieldBlox tracker account
- The naming convention for the utilization token's code is `u[underlying-issuer-code][asset-overlap-code][underlying-asset-code]`

#### Governance Token
YieldBlox uses a token-based governance model. Users receive governance tokens for participating in the YieldBlox protocol by lending or borrowing. They can then use these tokens to create governance proposals that modify the YieldBlox protocol and vote on governance proposals. For information on the YieldBlox governance system, see our docs page: https://docs.yieldblox.com/#/

**Token Identification:**
- Governance tokens are issued by the YieldBlox lending pool account
- The asset code for governance tokens is `YBX`

### Protocol Accounts
#### YieldBlox Lending Pool
The YieldBlox Lending Pool holds all assets deposited by lenders and lends them out to borrowers. It also issues pool tokens to lenders and governance tokens to lenders and borrowers. Finally, it stores critical protocol information in its data entries.

**Account Data Entries**
1. *Asset Data*: Stores underlying asset information. Seperate data entry for every asset supported by the pool
    - Data entry key: `[underlying-issuer-code][asset-overlap-code][underlying-asset-code(first 9 characters)]_Data`
    - Data entry value: `[interest-rate-numerator]_[utilization-factor]_[utilization-addend]_[liquidation-factor]_[liquidation-incentive]_[governance-allocation]_[last-3-asset-code-characters]`
        - Notes:
            1. If the asset is not permitted to be used as collater the liquidation factor and liquidation incentive characters will be `NA`
            2. If the asset is not permitted to be lent out the interest rate numberator, utilization factor, and utilization addend characters will be `NA`
            3. If the asset is less than 10 characters long the last 3 asset code characters will be blank, governance allocation will be the end of the data entry value.
2. *Asset Issuers*: Storying asset issuer account IDs
    - Data entry key: `[underlying-issuer-code][asset-overlap-code][underlying-asset-code(first 9 characters)]_Issuer`
    - Data entry value: `[underlying-asset-issuer-public-key]`

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
This account sends utilization token payments to the pool account when the utilization ratio increases and recieves them from the pool when the utilization rate decreases. These payments are aggregated to calculate the average utilization ratio for a loan or the utilization rate at the time a loan was originated.

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
While the User Account is not a protocol account in the sense that the protocol controls it, it still plays a critical role by sponsoring claimable balances that track the user's collateral and liabilities.

**Claimable Balances**
- *Collateral Claimable Balance*: The Mint txFunction creates this claimable balance for users to provide collateral to the protocol when they borrow from the lending pool. Users can deposit additional collateral at any time by creating more claimable balances. The claimable balances must be structured to be claimable by the pool at any time and claimable by the user in 100 years. When a user's loan is repaid or liquidated, the pool claims the claimable balance and either returns it to the user (if the loan was properly repayed), or confiscates it and pays it to the loan liquidator (if the loan was liquidated).
- *Liability Claimable Balances*: Used to track users liabilities. When a user borrows from the lending pool, the lending pool creates a claimable balance of liability tokens equal to the number of tokens borrowed. The pool can claim this claimable balance at any time, and the user can claim it in 100 years. This claimable balance structure allows the protocol to easily discover a user's liabilities and the amount of time the liabilities have been outstanding. The claimable balance is deleted when the user repays the loan or when the loan is liquidated. 

### Lending txFunctions
This section provides a brief overview of the txFunctions involved in the YieldBlox lending protocol. The full txFunctions will be publicly available on the YieldBlox GitHub repository once we officially launch our beta. If you would like more technical details on the txFunctions, feel free to contact us.
1. *Mint txFunction*\
The Mint txFunction is used to both lend assets to the pool in exchange for pool tokens and borrow from the pool by providing collateral. To lend assets, users deposit the assets they wish to lend in the lending pool and receive pool tokens proportional to the percentage of the pool's balance of the deposited asset. To borrow assets, the user creates a claimable balance of pool tokens that act as collateral for the assets they wish to borrow and withdraws the assets they wish to borrow from the pool. If they provide normal assets as collateral, the Mint txFunction converts the assets into pool tokens before locking them in a claimable balance. Since pool tokens are used as collateral, the user receives interest on their collateral deposit.\
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
The Liquidate txFunction is used to liquidate a delinquent accounts loans. Account can be liquidated when the borrowing account's health factor falls below one. The liquidator repays the delinquent accounts loans until the accounts health factor rises above one and is sent the value of their repayment in collateral along with a small liquidation incentive.
    - Calculations used:
        1. Asset Payout: Used to value the pool tokens used collateral
        2. Interest Accrued
        3. Liquidation Penalty
5. *Governance txFunction*\
The Governance txFunction is used to modify the protocol in any way, from tweaking the interest rate calculation to modifying the core txFunctions. Governance is a three-stage txFunction. 
    1. First Stage: The first stage creates a governance proposal by creating a proposal account that contains the proposed update's transaction hash as a signer. Users vote on the proposal by adding trustlines for `YES:proposalAccount` or `NO:proposalAccount assets`. 
    2. Second Stage: At the end of a 3 day period, the votes are tallied by running the Governance txFunction again and providing the proposal account's public key. Each user's vote is worth the number of governance tokens that their account holds. If a proposal passes, then the proposal account marks itself as approved using an account data entry. If it fails the proposal account is deleted.
    3. Third Stage: After 2 days have passed, the governance txFunction is ran a third time and provided the proposal account's public key to carry out the third stage of the txFunction. This stage signs the proposed update's transaction hash approving it to be submitted.  
6. *swapRate txFunction*\
This txFunction is used by borrowers to swap between fixed and floating rates. It can also be used by any protocol participant to rebalance a borrowers stable rate if it falls too far below the market rate. 
7. *Flash txFunction*\
This txFunction allows users to take out flash loans. Users can borrow any amount of assets from the lending pool as long as they repay the assets in the same transaction envelope that they borrow them in. This txFunction will not be turned on initially.

### Protocol Calculations
#### Utilization Ratio Calculations
##### Pool Utilization Ration
Used to calculate the current utilization ratio for an asset

![\Large](https://latex.codecogs.com/svg.latex?U%3D%5Cfrac%7BL%7D%7BL&plus;B%7D)

*U* = Current Utilization Ratio\
*L* = Total Liability Tokens. Tracked by YBX Tracker\
*B* = Total Pool Balance

##### Utilization Delta
Used to track the overall utilization delta of a transaction. This allows the protocol to measure an assets average utilization ratio over a period of time.

![\Large](https://latex.codecogs.com/svg.latex?U_d%3D%5Cfrac%7BL_i-L_%7Bi-1%7D%7D%7B%28L_i-L_%7Bi-1%7D%29&plus;%28B_i-B_%7Bi-1%7D%29%7D*%28b_%7Bi-1%7D-b_%7Bi-2%7D%29)

*L<sub>i</sub>*= Total liability tokens at the ledger state of the current transaction\
*L<sub>i-1</sub>*= Total liability tokens at the ledger state of the last utilization-modifying transaction\
*B<sub>i</sub>*=Total pool balance at the ledger state of the current utilization-modifying transaction\
*B<sub>i-1</sub>*= Total pool balance at the ledger state of the last utilization-modifying transaction\
*b<sub>i-1</sub>*=Block the last utilization-modifying transaction was applied in\
*b<sub>i-2</sub>*= Block the second to last utilization-modifying transaction was applied in

##### Utilization Adjustment
Used to calculate the necessary utilization ratio delta adjustment 

![\Large](https://latex.codecogs.com/svg.latex?U_a%20%3D%20%5Csum_%7Bi%3DT_%7B\Delta%20l%7D%7D%5E%7BT_%7B\Delta%20w%7D%7D%5Cfrac%7BL_%7Bi-2%7D%7D%7BL_%7Bi-2%7D+B_%7Bi-2%7D%7D*%28b_%7Bi-1%7D-b_%7Bi-2%7D%29%29-U_%7Bdi%7D-U_%7Bai%7D)

*L<sub>i-3</sub>*= Total liability tokens at the ledger state of the utilization-modifying transaction 3 utilization-modifying transactions ago\
*B<sub>i-3</sub>*= Total pool balance at the ledger state of the utilization-modifying transaction 3 utilization-modifying transactions ago\
*b<sub>i-2</sub>*=Block the second to last utilization-modifying transaction was applied in\
*b<sub>i-3</sub>*= Block the third to last utilization-modifying transaction was applied in\
*U<sub>di</sub>*= Utilization delta payment of the last utilization-modifying transaction\
*U<sub>ai</sub>*= Utilization adjustment payment of the last utilization-modifying transaction\
*U<sub>∆w</sub>*= Farthest back incorrect utilization delta payment\
*U<sub>∆l</sub>*= Utilization delta payment of the last utilization-modifying transaction\
##### Aggregated Utilization Ratio
Used to calculate the aggregated utilization ratio for a loan based on the amount of time it has been outstanding.

![\Large](https://latex.codecogs.com/svg.latex?U_a%20%3D%20%5Cfrac%7BB_c-B_o+U_a%7D%7Bb_i-b_o%7D+%5cfrac%7BU%7D%7Bb_c-b_i%7D)

*B<sub>c</sub>*= The current utilization tracker balance\
*B<sub>o</sub>*= The utilization tracker balance at the time of loan origination\
*U<sub>a</sub>*=The utilization ratio adjustment\
*b<sub>i</sub>*=Block of the last utilization delta payment\
*b<sub>o</sub>*=Block at loan origination\
*b<sub>c</sub>*=Current block\
*U*=Current utilization ratio


#### Interest Rate Calculations
YieldBlox uses a purely demand-based interest rate calculation, which means the only protocol variable involved in the equation is the utilization rate. This equation allows the protocol to more efficiently adjust for market conditions without changing a base interest rate. It also gives governance participants a large amount of flexibility when modifying protocol interest rates. The equation involves the Interest Numerator, Utilization Addend, and Utilization Factor constants which can be updated to change not only the slope of the interest rate curve, but also the curve's exponentiality and y-intercept. These constants are currently 10, 1.6, and -0.45, respectively, which results in the following curve.

 ![apr](_media/ybxInterestRates.png "Interest Rate Curve")

##### Interest Rate
Used to calculate the current interest rate.

![\Large](https://latex.codecogs.com/svg.latex?I%20%3D%5Cfrac%7Ba%7D%7B1&plus;%2810e%29%5E%7Bb&plus;c*U_r%7D%7D)

*I* = Interest Rate\
*U<sub>r</sub>* = Utilization Ratio\
*a* = Interest Numerator. Set by pool\
*b* = Utilization Addend. Set by pool\
*c* = Utilization Factor. Set by pool

##### Interest Accrued
Used to calculate the interest fees accrued by a loan.

![\Large](https://latex.codecogs.com/svg.latex?I_a%20%3DA*%5Cfrac%7Ba%7D%7B1&plus;%2810e%29%5E%7Bb&plus;c*U_a%7D%7D)

*I<sub>a</sub>* = Accrued interest\
*A* = Loan amount\
*U<sub>a</sub>* = Aggregated Utilization Ratio\
*a* = Interest Numerator. Set by pool\
*b* = Utilization Addend. Set by pool\
*c* = Utilization Factor. Set by pool

##### Stable Rate
Used to calculate the stable interest rate for a loan

![\Large](https://latex.codecogs.com/svg.latex?I_a%20%3D%5Cfrac%7Ba%7D%7B1&plus;%2810e%29%5E%7Bb&plus;c*U_a%7D%7D+I)

*I<sub>s</sub>*= Stable rate
*U*= Utilization ratio at the time the loan was originated
*U<sub>o</sub>*= Optimal utilization ratio. Set by a pool data entry. Initially set at .83. Equal to the point at which the formulaic interest rate begins increasing at a rate greater than the ¼ the rate that the utilization ratio increases at.
*a*= Interest Numerator. Set by a pool data entry. Initially set at 10
*b*= Utilization Addend. Set by a pool data entry. Initially set at 1.6
*c*= Utilization Factor. Set by a pool data entry. Initially set at -0.45
*I*= Current interest rate

#### Min Collateral Requirement
Used to calculate the Minimum Collateral Requirement for a loan.

![\Large](https://latex.codecogs.com/svg.latex?V_c%20%3D%20%5Cfrac%7BV_l*1.01%7D%7B%5Coverline%7BF%7D%7D)

*V<sub>C</sub>* = Minimum collateral value requirement\
*C<sub>l</sub>* = Loan value\
*F̅* = Average liquidation factor of the selected collateral types

#### Maximum Liquidation Amount
Used to calculate the maximum amount of a  loans value the liquidator is allowed to liquidate in order to reach a health factor of 1.01

![\Large](https://latex.codecogs.com/svg.latex?\Delta%20V_l%20%3D%20%5Cfrac%7B%5Coverline%7BF_a%7D*V_c-1.01*V_l%7D%7B%5Coverline%7BI%7D*%5Coverline%7BF_w%7D-1.01%7D)

*∆V<sub>l</sub>*= max allowable liquidation amount\
*V<sub>c</sub>*= Collateral value\
*F̅<sub>a</sub>*= Average liquidation factor for the accounts collateral deposits\
*V<sub>l</sub>*= Liability value\
*I̅*= Average liquidation incentive for the collateral assets being withdrawn\
*F<sub>w</sub>*= Average liquidation factor for the collateral assets being withdrawn

#### Health Factor
Used to calculate an accounts health factor

![\Large](https://latex.codecogs.com/svg.latex?V_c%20%3D%20%5Cfrac%7B%5Csum_%7Bi%3DC_0%7D%5E%7B%7CC%7C%7D%28F_i*V_%7Bci%7D%29%7D%7B%5Csum_%7Bi%3DL_0%7D%5E%7B%7CL%7C%7DV_%7Bli%7D%7D)

*C<sub>0</sub>*= First deposited collateral asset\
*|C|*= Last deposited collateral asset\
*F<sub>i</sub>*= Liquidation factor for collateral asset i\
*V<sub>ci</sub>*= Collateral value of collateral asset i\
*L<sub>0</sub>*= First outstanding loan\
*|L|*= Last outstanding loan\
*V<sub>ci</sub>*= Liability value of loaned asset i

#### Maximum Liability
Used to calculate the maximum liability an account can hold at one time

![\Large](https://latex.codecogs.com/svg.latex?V_c%20%3D%20%5Cfrac%7B%5Csum_%7Bi%3DC_0%7D%5E%7B%7CC%7C%7D%28F_i*V_%7Bci%7D%29%7D%7B1.01%7D)

*V<sub>l</sub>*= Max liability value for the account\
*C<sub>0</sub>*= First deposited collateral asset\
*|C|*= Last deposited collateral asset\
*F<sub>i</sub>*= Liquidation factor for collateral asset i\
*V<sub>ci</sub>*= Collateral value of collateral asset i\


#### Pool Tokens Issued
Used to calculate the number of pool tokens issued to a lender.

![\Large](https://latex.codecogs.com/svg.latex?O_i%20%3D%20%5Cfrac%7B%5Cfrac%7BR_n%7D%7BR_c&plus;L%7DO_o%7D%7B1-%5Cfrac%7BR_n%7D%7BR_c&plus;L%7D%7D)

*O<sub>i</sub>* = New pool tokens that will be issued to the lender\
*O<sub>o</sub>* = Current outstanding pool tokens\
*R<sub>n</sub>* = Assets the user is adding to the lending pool\
*R<sub>c</sub>* = Current pool reserves of the asset the user is lending\
*L*= Current outstanding liability tokens for the asset the user is lending

#### Asset payout
Used to calculate the asset payout for a given number of burned pool tokens.

![\Large](https://latex.codecogs.com/svg.latex?A%20%3D%20%28B&plus;L%29%5Cfrac%7BT_b%7D%7BT_t%7D)

*A*= Amount of the asset paid out\
*B* = Pool balance of the underlying asset of the pool token being burned\
*L* = Total liability tokens outstanding for the underlying asset of the pool token being burned\
*T<sub>b</sub>* = Number of pool tokens burned\
*T<sub>t</sub>* = Total number of pool tokens




