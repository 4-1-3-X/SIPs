---
sip: 59
title: Futures reference price methodology
status: WIP
author: Bill Mayott <bill.mayott@xbto.com>, Philippe Bekhazi <philippe@xbto.com>, Walton Comer <walton@xbto.com>, Kain Warwick (@kaiynne)
discussions-to: #sips-wips

created: 2020-05-18
---

## Simple Summary
<!--"If you can't explain it simply, you don't understand it well enough." Provide a simplified and layman-accessible explanation of the SIP.-->
A mechanism to convert prices from futures markets into a reference prices for Synths.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->
This SIP describes a general approach to coverting pricing data from futures markets into a single reference price, as well as a specific approach methodology for creating a non-expiring Crude Oil Index based on CME Light Sweet Crude Oil futures prices (Contract Code: CL) for a Synth with ticker symbol sOIL. There are many markets where price discovery occurs solely or predominantly in the futures markets, these futures markets are made up of individual contracts with varying expiry dates. This presents a problem in creating a sungle reference price to be published on Ethereum for creating a synthetic asset. The solution proposed in this SIP is to employ a dynamic weighting scheme (in continuous time) of the near two contract months, with emphasis initially given to the near contract.  As expiry approaches, however, weight is progressively shifted out of the near contract in favor of the 2nd month.  Moreover, upon reaching 5 days (configurable via SCCP) prior the last trade time (2:30PM EST on the exchange stipulated Last Trade day), zero weighting in the near contract is achieved, with the weight instead being allocated between the 2nd and 3rd contract months.  Once the front month expires, the next nearest two live contracts become the 1st and 2nd months, and the dynamic weighting process repeats. This weighting methodology is intentionally relatively simple and linear for easier replication.

## Motivation
<!--The motivation is critical for SIPs that want to change Synthetix. It should clearly explain why the existing protocol specification is inadequate to address the problem that the SIP solves. SIP submissions without sufficient motivation may be rejected outright.-->
The creation of a single reference price for futures markets that incorporates information from the most liquid futures contracts enables assets like WTI and other commodities to be traded as synthetic assets within Synthetix. There is significant demand for these assets particularly given the liquidity of the underlying markets and the difficulty of access for the average trader. 
 
## Specification
<!--The technical specification should describe the syntax and semantics of any new feature.-->

The forumla for the reference price is below:
price = \begin{cases}
\frac{d_{1} \ - \ X}{d_{1} \ - \ d_{0}} \cdot P_1 \ + \ \frac{d_{0} \ + \ X}{d_{1} \ - \ d_{0}} \cdot P_2 &   \ \mbox{if } \ \ \  X \leq d_1 \\
\\
\frac{d_{2} \ - \ X}{d_{2} \ - \ d_{1}} \cdot P_2 \ + \ \frac{X \ - \ d_{1}}{d_{2} \ - \ d_{1}} \cdot P_3 & \ \mbox{if } \ \ \ 0 \lt d_1 \lt X \\

\end{cases}

X: # of days prior to expiration to achieve zero weight in in the expiring contract
DTE0: Days since the prior month contract expired
DTE1: Days remaining for the current front month contact
DTE2: Days remaining for the current 2nd month contract
P0: Orderbook mid-price of the current front month contract
P1: Orderbook mid-price of the current 2nd month contract
p2: Orderbook mid-price of the current 3rd month contract

## Rationale
<!--The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->
This method was chosen over other more strictly enforced constant-maturity methods, such as cubic spline best-fit regressions of the futures curve given the added complexity of these methods and the marginal difference in accuracy of the reference price. The goal of this SIP is to enable broader understandability, adoption and redundancy for a variety of assets for which price discovery is predominantly or solely centred around futures contract trading. By providing a straightforward methodology we can construct reference prices for a range of assets that can be published by Chainlink Oracles onto Ethereum bridging TradFi and DeFI.

## Test Cases
<!--Test cases for an implementation are mandatory for SIPs but can be included with the implementation..-->
The logic of these reference prices is implemented at the data provider level, with Chainlink node operators consuming this data and publishing it into agregator contracts. In order to ensure the reliability of this data multiple data providers will be selected to feed the node operators all of which connect dirctly to raw data from the futures markets.

## Implementation
<!--The implementations must be completed before any SIP is given status "Implemented", but it need not be completed before the SIP is "Approved". While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.-->
The implementations must be completed before any SIP is given status "Implemented", but it need not be completed before the SIP is "Approved". While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.

## Configurable Values (Via SCCP)
The primary configurable variable in this SIP is the number of days to expiry defined as X in the formula above.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).