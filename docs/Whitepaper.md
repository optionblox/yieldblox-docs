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

The YieldBlox protocol serves as a tier-3 blockchain app, a layer between YieldBlox web-app users and the Stellar ledger. Users link their wallets to YieldBlox's web-app, which takes user inputs and communicates them to the YieldBlox protocol. The protocol uses TSS txFunctions to build and sign Stellar transactions that carry out protocol lending operations. Users approve these transactions with their wallet and receive the result of their transaction, whether that's pool tokens or a loan.

### High-Level Protocol Diagram

![protocol](_media/ybxProtocolHighLevel.png "YBX Protocol")

## Protocol Structure

The YieldBlox protocol facilitates lending by utilizing a network of protocol accounts, tokens, and claimable balances.

### YieldBlox Governance

YieldBlox is completeley decentralized in that the core team does not retain control over the protocol after it is released. The protocol uses a decentralized governance token model to make changes to the protocol. YieldBlox distributes a Stellar-based governance token called YBX to protocol users. This tokens is used to control the protocol by voting on proposed protocol changes using the governance txFunction. The result of this system is the control and future of the protocol are purely in the hands of its users.

### Loan Terms

Loans can be taken out for any time period as long as the user maintains an account health factor above 1 (an accounts health factor is based on the accounts total liability value, collateral value, and collateral liquidation factors). If the accounts health factor falls below 1, the accounts loans can be liquidated by another protocol participant until their account health factor increases to 1.01.

### Loan Interest Rates

Users can borrow from YieldBlox at either fixed or floating rates. Floating rates are based purely on demand and will fluctuate based on the borrowed assets utilization ratio. Fixed rates are a multiple of the floating rate at loan origination. They will always be higher than floating rates but some borrowers may prefer the stability of fixed rates.

#### Rate Swapping

A loans fixed rate can be force rebalanced up to the current stable rate if the loans rate has fallen below the market floating rate. This is accomplished with the swapRate txFunction. User's can also use the swapRate txFunction to swap their loan from fixed to the current floating rate or from floating to the current fixed rate.

#### Interest Rate Distribution

90% of interest rate fees are sent directly to the pool to distribute them to protocol lenders. Lenders will withdraw these fees when they burn their pool tokens. The remaining 10% of fees are used to repurchase and YBX tokens on the Stellar DEX. This repurchasing is necessary to support the YBX backstop which we will go over in the next section.

### Collateralization and Liquidation

#### Definitions

Before we discuss loan collateralization and position liquidation we must define _"Health Factor_", _"Liquidation Factor_", _"Liquidation Incentive_". The first of these is _"Health Factor"_. An accounts health factor is a measure of it's collateralization levels. To originate a loan a user must ensure that their health factor will be above 1.2 after loan origination. If a users account health factor drops below one their positions can be liquidated until their health factor reaches 1.01. Health factor is calculated with the users liability(outstanding loan value+accrued interest) value, collateral value, average collalteral liquidation factor, and average collateral liquidation incentive. Liquidation factors are assigned to supported assets by the protocol, they govern the point at which a position the asset is collateralizing can be liquidated. Liquidation incentives are also assigned to supported assets by the protocol, they govern the discount liquidators will recieve when withdrawing a collateralized asset from the protocol during a liquidation.

#### Collateralization

Whenever a user borrows from the YieldBlox protocol they must provide collateral by allocating pool lending deposits to act as collateral. They must deposit sufficient collateral to keep their health factor above 1.2 and if their health factor ever drops below 1 their positions can be liquidated. Collateralized assets are converted into pool tokens so that they can be lent out while serving as collateral. This allows borrowers to generate interest on their collateral deposits. It should be noted that this does not increase protocol risk. In the case of a liquidation, liquidators will withdraw the pool token rather than it's associated underlying asset.

#### Liquidation

Borrowers can be liquidated if their account health factor drops below 1. This means that another protocol user can call the liquidation txFunction and repay a portion of the borrowers debt in exchange for a portion of the borrowers collateral. The borrower can only be liquidated to the point where their health factor reaches 1.01. This means borrowers are unlikely to have their position fully liquidated. To incentivize liquidations of undercollateralized positions, liquidators recieve a discount on the assets they withdraw from the borrowers collateral based on the collateralized assets liquidation incentive. For example, if a liquidator repays 100 USD of a borrowers position and the borrowers collateral has a liquidation incentive of 1.05, the liquidator is permitted to withdraw 105 USD worth of the collateralized asset.

#### YBX Backstop

In cases of extreme volatility it is possible for the liquidation incentive to be impossible to pay out under normal market conditions. This could happen if the value of the users liability increased drastically in a very short amount of time or the value of the users collateral suddenly plunged. To ensure the undercollateralized loan is still liquidated YieldBlox uses the YBX backstop system. If the liquidation txFunction is ran to liquidate a position that cannot provide the liquidation incentive due to market conditions, the txFunction will recognize this with its price feed and reduce the amount of the loan the liquidator is expected to repay to the point where the liquidation incentive can be withdrawn. The txFunction will then repay the remaining portion of the loan by minting new YBX and selling it on the Stellar DEX for the required repayment asset and amount. This is done using a pathPayment operation. To compensate for this backstop 10-20% of interest rate fees are used to re-purchase and burn YBX on the DEX. The best way to think about this system is the protcol issuing debt to YBX holders to cover uncollectable user-debt and repurchasing this debt over time through YBX buybacks.

### Protocol Data Feeds

The YieldBlox protocol relies on utilization and price feeds to function.

#### Utilization Feed

The protocol tracks its utilization over time with a YieldBlox Tracker account. Whenever a protocol transaction increases an assets utilization ratio the tracker account sends utilization tokens to the pool account. Whenever a protocol transaction decreases an assets utilization ratio the pool account sends utilization tokens back to the tracker account and the tracker account tracks this action in its operations history by sending an identical payment to itself. This process is necessary to allow the protocol to determine the average utilization rate over a loan's lifecycle, which is required to calculate a loan's accrued interest fees. The amount of tokens sent reflects the utilization delta of the payment before last multiplied by the number of blocks between the secont to last and last utilization modifying transaction.

For example, given the following ledger state:

1. Second to last utilization modifying transaction: Occured on block 1000 and Borrowed 100 XLM increasing the utilization ratio by .1
2. Last utilization modifying transaction: Occured on block 1100, the utilization delta of this transaction doesn't matter
3. Current transaction: Will include a utilization delta payment of 10 utilization tokens (100 blocks \* .1 utilization delta)

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

The YieldBlox tracker account has clawback enabled so that it can clawback this asset.

**Token Identification:**

- Utilization tokens are issued by the YieldBlox tracker account
- The naming convention for the utilization token's code is `u[underlying-issuer-code][asset-overlap-code][underlying-asset-code]`

#### Governance issuance tracker token

Governance issuance tracker tokens are used to track the issuance that is allocated to a specific governance token on a givern issuance period. At the beginning of each issuance period a payment is made from the governance issuance tracker account to the governance issuance tracker account equal to the amount of YBX tokens to be paid out for each pool token of that kind.

**Token Identification:**

- Governance issuance tracker tokens are issued by the governance issuance tracker account
- The naming converntion for the token is `[y/l][underlying-issuer-code][asset-overlap-code][underlying-asset-code]`

#### Governance Token

YieldBlox uses a token-based governance model. Users receive governance tokens for participating in the YieldBlox protocol by lending or borrowing. They can then use these tokens to create governance proposals that modify the YieldBlox protocol and vote on governance proposals. For information on the YieldBlox governance system, see our docs page: https://docs.yieldblox.com/#/

**Token Identification:**

- Governance tokens are issued by the YieldBlox tracker account
- The asset code for governance tokens is `YBX`

### Protocol Accounts

#### YieldBlox Lending Pool

The YieldBlox Lending Pool holds all assets deposited by lenders and lends them out to borrowers. It also issues pool tokens to lenders and governance tokens to lenders and borrowers. Finally, it stores critical protocol information in its data entries.

**Account Data Entries**

1. _Asset Data_: Stores underlying asset information. Seperate data entry for every asset supported by the pool
   - Data entry key: `[underlying-issuer-code]_[asset-overlap-code]_[underlying-asset-code(first 9 characters)]_Data`
   - Data entry value: `[exponential-constant]_[base-rate-constant]_[peak-rate-constant]_[liquidation-factor]_[liquidation-incentive]_[lending-governance-allocation]_[borrowing-governance-allocation]_[YBX-Fee-allocation]_[last-3-asset-code-characters]`
     - Notes:
       1. If the asset is not permitted to be used as collater the liquidation factor and liquidation incentive characters will be `NA`
       2. If the asset is not permitted to be lent out the interest rate numberator, utilization factor, and utilization addend characters will be `NA`
       3. If the asset is less than 10 characters long the last 3 asset code characters will be blank, YBX fee allocation will be the end of the data entry value.
2. _Asset Issuers_: Stores asset issuer account IDs
   - Data entry key: `[underlying-issuer-code]_[asset-overlap-code]_[underlying-asset-code(first 9 characters)]_Issuer`
   - Data entry value: `[underlying-asset-issuer-public-key]`
3. _Contract Frozen_: Records whether contracts are frozen or not - Data entry key: `frozen_contracts` - Data entry value: `[contract-name]-[T/F]_[contract-name]-[T/F]...`
   **Trustlines**

- All supported assets

**Signature structure**\
_Thresholds_

- Low Threshold: 20
- Medium Threshold:20
- High Threshold:30

_Signers_

- Burn txFunction: 3 signers, weight 10
- Mint txFunction: 3 signers, weight 10
- Repay txFunction: 3 signers, weight 10
- Liquidate txFunction: 3 signers, weight 10
- Flash txFunction: 3 signers, weight 10
- Governance txFunction: 5 signers, weight 10

#### Governance Issuance Tracker

Thhis account sends governance issuance tokens to itself each governance issuance period. The payment amounts are equal to the number of governance tokens that should be issued for 1 pool token. These payments are used to calculate how many governance tokens should be issued to a user based on their holdings at that time.

**Trustlines**

- Governance Issuance Tracker totken

**Signature Structure**

- normal protocol signers

_Flags_

- AuthRevokable
- ClawbackEnabled

#### YieldBlox Tracker

This account sends utilization token payments to the pool account when the utilization ratio increases and recieves them from the pool when the utilization rate decreases. These payments are aggregated to calculate the average utilization ratio for a loan or the utilization rate at the time a loan was originated.

**Trustlines**

- Utilization token
- Liability token

**Signature structure**\
_Thresholds_

- Low Threshold: 20
- Medium Threshold:20
- High Threshold:30

_Signers_

- Repay txFunction: 3 signers, weight 10
- Liquidate txFunction: 3 signers, weight 10
- Governance txFunction: 5 signers, weight 10

_Flags_

- AuthRevokable
- ClawbackEnabled

#### User Account

While the User Account is not a protocol account in the sense that the protocol controls it, it still plays a critical role by sponsoring claimable balances that track the user's collateral and liabilities.

**Claimable Balances**

- _Collateral Claimable Balance_: The Mint txFunction creates this claimable balance for users to provide collateral to the protocol when they borrow from the lending pool. Users can deposit additional collateral at any time by creating more claimable balances. The claimable balances must be structured to be claimable by the pool at any time and claimable by the user in 100 years. When a user's loan is repaid or liquidated, the pool claims the claimable balance and either returns it to the user (if the loan was properly repayed), or confiscates it and pays it to the loan liquidator (if the loan was liquidated).
- _Liability Claimable Balances_: Used to track users liabilities. When a user borrows from the lending pool, the lending pool creates a claimable balance of liability tokens equal to the number of tokens borrowed. The pool can claim this claimable balance at any time, and the user can claim it in 100 years. This claimable balance structure allows the protocol to easily discover a user's liabilities and the amount of time the liabilities have been outstanding. The claimable balance is deleted when the user repays the loan or when the loan is liquidated.

### Lending txFunctions

This section provides a brief overview of the txFunctions involved in the YieldBlox lending protocol. The full txFunctions will be publicly available on the YieldBlox GitHub repository once we officially launch our beta. If you would like more technical details on the txFunctions, feel free to contact us.

1. _Mint txFunction_\
   The Mint txFunction is used to both lend assets to the pool in exchange for pool tokens and borrow from the pool by providing collateral. To lend assets, users deposit the assets they wish to lend in the lending pool and receive pool tokens proportional to the percentage of the pool's balance of the deposited asset. To borrow assets, the user creates a claimable balance of pool tokens that act as collateral for the assets they wish to borrow and withdraws the assets they wish to borrow from the pool. If they provide normal assets as collateral, the Mint txFunction converts the assets into pool tokens before locking them in a claimable balance. Since pool tokens are used as collateral, the user receives interest on their collateral deposit.\
   - Calculations Used: 1. Minimum Collateral Requirement 2. Pool Tokens Issued
2. _Burn txFunction_\
   The Burn txFunction is used to convert pool tokens into underlying assets. Users burn their pool tokens and receive the assets associated with the pool tokens. - Calculations Used: 1. Asset payout
3. _Repay txFunction_\
   The Repay txFunction is used to repay loans. Users send the repayment to the pool and receive their collateral balance net of accrued interest fees. - Calculations Used: 1. Interest Accrued
4. _Liquidate txFunction_\
   The Liquidate txFunction is used to liquidate a delinquent accounts loans. Account can be liquidated when the borrowing account's health factor falls below one. The liquidator repays the delinquent accounts loans until the accounts health factor rises above one and is sent the value of their repayment in collateral along with a small liquidation incentive. - Calculations used: 1. Asset Payout: Used to value the pool tokens used collateral 2. Interest Accrued 3. Liquidation Penalty
5. _Governance txFunction_\
   The Governance txFunction is used to modify the protocol in any way, from tweaking the interest rate calculation to modifying the core txFunctions. Governance is a three-stage txFunction. 1. First Stage: The first stage creates a governance proposal by creating a proposal account that contains the proposed update's transaction hash as a signer. Users vote on the proposal by adding trustlines for `YES:proposalAccount` or `NO:proposalAccount assets` and creating manageSellOffers selling their YBX tokens for YES or NO tokens at a price of 1. 2. Second Stage: At the end of a 3 day period, the votes are tallied by running the Governance txFunction again and providing the proposal account's public key. Each user's vote is worth the number of governance tokens that their account holds. If a proposal passes, then the proposal account marks itself as approved using an account data entry. If it fails the proposal account is deleted. 3. Third Stage: After 2 days have passed, the governance txFunction is ran a third time and provided the proposal account's public key to carry out the third stage of the txFunction. This stage signs the proposed update's transaction hash approving it to be submitted.

   - Alternate Governance txFunction operations - _revokeSigner_\
     This command allows governance token holders to immediatley remove a signer from the protocol accounts and replace it with a different one. This operation can only be ran twice per day. This is a 2 stage operation. 1. First Stage: The first stage creates a proposal account that is labeled a revokeSigner operation. The proposal account will also contain a data entry with the proposed transaction hash. Users vote for the proposal with the method outlined previously. 2. Second Stage: As soon as 15% of governance token holders have voted yes the second stage of the operation can occur. In this stage the proposal is immediatley implemented by signing the proposed transaction hash. - _lockProtocol_\
     This command allows governance token holders to lock any specified protocol operations besides the Governance txFunction. They can be unlocked using another lockProtocol command or a governanceProposal command. This is also a 2 stage operation. 1. First Stage: The first stage creates a proposal account that is labeled a revokeSigner operation. The proposal account will also contain a data entry with the proposed transaction hash. Users vote for the proposal with the method outlined previously. 2. Second Stage: As soon as 15% of governance token holders have voted yes the second stage of the operation can occur. In this stage the proposal is immediatley implemented by signing the proposed transaction hash.

6. _swapRate txFunction_\
   This txFunction is used by borrowers to swap between fixed and floating rates. It can also be used by any protocol participant to rebalance a borrowers stable rate if it falls too far below the market rate.
7. _Flash txFunction_\
   This txFunction allows users to take out flash loans. Users can borrow any amount of assets from the lending pool as long as they repay the assets in the same transaction envelope that they borrow them in. This txFunction will not be turned on initially.

### Protocol Calculations

#### Utilization Ratio Calculations

##### Pool Utilization Ration

Used to calculate the current utilization ratio for an asset

![\Large](https://latex.codecogs.com/svg.latex?U%3D%5Cfrac%7BL%7D%7BL+B%7D)

_U_ = Current Utilization Ratio\
_L_ = Total Liability Tokens. Tracked by YBX Tracker\
_B_ = Total Pool Balance

##### Utilization Delta

Used to track the overall utilization delta of a transaction. This allows the protocol to measure an assets average utilization ratio over a period of time.

![\Large](https://latex.codecogs.com/svg.latex?U_d%3D%5Cfrac%7BL_i-L_%7Bi-1%7D%7D%7B%28L_i-L_%7Bi-1%7D%29+%28B_i-B_%7Bi-1%7D%29%7D*%28b_%7Bi-1%7D-b_%7Bi-2%7D%29)

_L<sub>i</sub>_= Total liability tokens at the ledger state of the current transaction\
_L<sub>i-1</sub>_= Total liability tokens at the ledger state of the last utilization-modifying transaction\
_B<sub>i</sub>_=Total pool balance at the ledger state of the current utilization-modifying transaction\
_B<sub>i-1</sub>_= Total pool balance at the ledger state of the last utilization-modifying transaction\
_b<sub>i-1</sub>_=Block the last utilization-modifying transaction was applied in\
_b<sub>i-2</sub>_= Block the second to last utilization-modifying transaction was applied in

##### Utilization Adjustment

Used to calculate the necessary utilization ratio delta adjustment

![\Large](https://latex.codecogs.com/svg.latex?U_a%20%3D%20%5Csum_%7Bi%3DT_%7B\Delta%20l%7D%7D%5E%7BT_%7B\Delta%20w%7D%7D%5Cfrac%7BL_%7Bi-2%7D%7D%7BL_%7Bi-2%7D+B_%7Bi-2%7D%7D*%28b_%7Bi-1%7D-b_%7Bi-2%7D%29%29-U_%7Bdi%7D-U_%7Bai%7D)

_L<sub>i-2</sub>_= Total liability tokens at the ledger state of the utilization-modifying transaction 3 utilization-modifying transactions ago\
_B<sub>i-2</sub>_= Total pool balance at the ledger state of the utilization-modifying transaction 3 utilization-modifying transactions ago\
_b<sub>i-2</sub>_=Block the second to last utilization-modifying transaction was applied in\
_b<sub>i-3</sub>_= Block the third to last utilization-modifying transaction was applied in\
_U<sub>di</sub>_= Utilization delta payment of the last utilization-modifying transaction\
_U<sub>ai</sub>_= Utilization adjustment payment of the last utilization-modifying transaction\
_U<sub>∆w</sub>_= Farthest back incorrect utilization delta payment\
_U<sub>∆l</sub>_= Utilization delta payment of the last utilization-modifying transaction\

##### Aggregated Utilization Ratio

Used to calculate the aggregated utilization ratio for a loan based on the amount of time it has been outstanding.

![\Large](https://latex.codecogs.com/svg.latex?U_a%20%3D%20%5Cfrac%7BB_c-B_o+U_a%7D%7Bb_i-b_o%7D+%5cfrac%7BU%7D%7Bb_c-b_i%7D)

_B<sub>c</sub>_= The current utilization tracker balance\
_B<sub>o</sub>_= The utilization tracker balance at the time of loan origination\
_U<sub>a</sub>_=The utilization ratio adjustment\
_b<sub>i</sub>_=Block of the last utilization delta payment\
_b<sub>o</sub>_=Block at loan origination\
_b<sub>c</sub>_=Current block\
_U_=Current utilization ratio

#### Interest Rate Calculations

YieldBlox uses an exponential demand-based interest rate calculation, which means the interest rate exponentially increases as the utilization ratio increases. This equation allows the protocol to more efficiently adjust for market conditions. It also gives governance participants a large amount of flexibility when modifying protocol interest rates. The equation involves Exponential, Base Interest Rate, and Peak Rate constants which can be updated to change not only the slope of the interest rate curve, but also the curve's exponentiality and y-intercept. These constants are currently 500, 0.05, and 11,000, respectively, which results in the following interest rate curves for floating and fixed rates.

![apr](_media/ybxInterestRates.png "Interest Rate Curve")

##### Interest Rate

Used to calculate the current interest rate.

![\Large](https://latex.codecogs.com/svg.latex?I%20%3D%5Cfrac%7B%280.0003%5E%7B-U%7D+a%29U%7D%7Bc%7D+b)

_I_ = Interest Rate\
_U<sub>r</sub>_ = Utilization Ratio\
_a_ = Exponential constant. Controls curve exponentiality. Set by a pool data entry.\
_b_ = Base interest rate constant. Controls base interest rate. Set by a pool data entry\
_c_ = Peak rate constant. Controls peak interest rate. Set by a pool data entry.

##### Interest Accrued

Used to calculate the interest fees accrued by a loan.

![\Large](https://latex.codecogs.com/svg.latex?I_a%20%3D%5Cfrac%7B%280.0003%5E%7B-U_a%7D+a%29U%7D%7Bc%7D+b)

_I<sub>a</sub>_ = Accrued interest\
_A_ = Loan amount\
_U<sub>a</sub>_ = Aggregated Utilization Ratio\
_a_ = Exponential constant. Controls curve exponentiality. Set by a pool data entry.\
_b_ = Base interest rate constant. Controls base interest rate. Set by a pool data entry\
_c_ = Peak rate constant. Controls peak interest rate. Set by a pool data entry.

##### Stable Rate

Used to calculate the stable interest rate for a loan

![\Large](https://latex.codecogs.com/svg.latex?I_a%20%3DI+I%281.05-U%29)

_I<sub>s</sub>_= Stable rate\
_U_= Utilization ratio at the time the loan was originated\
_I_= Current interest rate

#### Min Collateral Requirement

Used to calculate the Minimum Collateral Requirement for a loan.

![\Large](https://latex.codecogs.com/svg.latex?V_c%20%3D%20%5Cfrac%7BV_l*1.01%7D%7B%5Coverline%7BF%7D%7D)

_V<sub>C</sub>_ = Minimum collateral value requirement\
_C<sub>l</sub>_ = Loan value\
_F̅_ = Average liquidation factor of the selected collateral types

#### Maximum Liquidation Amount

Used to calculate the maximum amount of a loans value the liquidator is allowed to liquidate in order to reach a health factor of 1.01

![\Large](https://latex.codecogs.com/svg.latex?\Delta%20V_l%20%3D%20%5Cfrac%7B%5Coverline%7BF_a%7D*V_c-1.01*V_l%7D%7B%5Coverline%7BI%7D*%5Coverline%7BF_w%7D-1.01%7D)

_∆V<sub>l</sub>_= max allowable liquidation amount\
_V<sub>c</sub>_= Collateral value\
_F̅<sub>a</sub>_= Average liquidation factor for the accounts collateral deposits\
_V<sub>l</sub>_= Liability value\
_I̅_= Average liquidation incentive for the collateral assets being withdrawn\
_F<sub>w</sub>_= Average liquidation factor for the collateral assets being withdrawn

#### Health Factor

Used to calculate an accounts health factor

![\Large](https://latex.codecogs.com/svg.latex?V_c%20%3D%20%5Cfrac%7B%5Csum_%7Bi%3DC_0%7D%5E%7B%7CC%7C%7D%28F_i*V_%7Bci%7D%29%7D%7B%5Csum_%7Bi%3DL_0%7D%5E%7B%7CL%7C%7DV_%7Bli%7D%7D)

_C<sub>0</sub>_= First deposited collateral asset\
_|C|_= Last deposited collateral asset\
_F<sub>i</sub>_= Liquidation factor for collateral asset i\
_V<sub>ci</sub>_= Collateral value of collateral asset i\
_L<sub>0</sub>_= First outstanding loan\
_|L|_= Last outstanding loan\
_V<sub>ci</sub>_= Liability value of loaned asset i

#### Maximum Liability

Used to calculate the maximum liability an account can hold at one time

![\Large](https://latex.codecogs.com/svg.latex?V_c%20%3D%20%5Cfrac%7B%5Csum_%7Bi%3DC_0%7D%5E%7B%7CC%7C%7D%28F_i*V_%7Bci%7D%29%7D%7B1.01%7D)

_V<sub>l</sub>_= Max liability value for the account\
_C<sub>0</sub>_= First deposited collateral asset\
_|C|_= Last deposited collateral asset\
_F<sub>i</sub>_= Liquidation factor for collateral asset i\
_V<sub>ci</sub>_= Collateral value of collateral asset i\

#### Pool Tokens Issued

Used to calculate the number of pool tokens issued to a lender.

![\Large](https://latex.codecogs.com/svg.latex?O_i%20%3D%20%5Cfrac%7B%5Cfrac%7BR_n%7D%7BR_c+L%7DO_o%7D%7B1-%5Cfrac%7BR_n%7D%7BR_c+L%7D%7D)

_O<sub>i</sub>_ = New pool tokens that will be issued to the lender\
_O<sub>o</sub>_ = Current outstanding pool tokens\
_R<sub>n</sub>_ = Assets the user is adding to the lending pool\
_R<sub>c</sub>_ = Current pool reserves of the asset the user is lending\
_L_= Current outstanding liability tokens for the asset the user is lending

#### Asset payout

Used to calculate the asset payout for a given number of burned pool tokens.

![\Large](https://latex.codecogs.com/svg.latex?A%20%3D%20%28B+L%29%5Cfrac%7BT_b%7D%7BT_t%7D)

_A_= Amount of the asset paid out\
_B_ = Pool balance of the underlying asset of the pool token being burned\
_L_ = Total liability tokens outstanding for the underlying asset of the pool token being burned\
_T<sub>b</sub>_ = Number of pool tokens burned\
_T<sub>t</sub>_ = Total number of pool tokens

#### YBX Issuance

Used to calculate the amount of YBX that should be issued at a given time.

![\Large](https://latex.codecogs.com/svg.latex?I%20%3D%201.00000034430845^%7BT+25,000,000-O%7D)

_I_= YBX issued in this instance\
_T_ = Total YBX tokens to be issued\
_O_ = Total YBX tokens currently outstanding\

#### YBX Backstop mount

Used to calculate how much of the users liability should be repaid with the YBX backstop.

![\Large](https://latex.codecogs.com/svg.latex?R%3D%2%5Cfrac%7B%5Coverline%7BF_a%7D*V_c-1.01*V_l%7D%7B%5Coverline%7BI_a%7D*%5Coverline%7BF_a%7D-1.01%7D*%5coverline%7BI_a%7D-V_c)

_R_= Amount of the users liability that should be repaid with the YBX backstop
_V<sub>c</sub>_= Collateral value\
_F̅<sub>a</sub>_= Average liquidation factor for the accounts collateral deposits\
_V<sub>l</sub>_= Liability value\
_I̅_a_= Average liquidation incentive for the accounts collateral deposits
