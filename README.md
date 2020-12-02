# Overview

Rho is a protocol for peer-to-pool interest rate swaps. Swaps are benchmarked against cToken interest rates, meaning that users can either pay a fixed interest rate and receive the floating cToken borrow rate, or vice versa. The protocol uses the benchmarked cToken as collateral, so that users can earn interest while their collateral is locked. The accrued COMP goes into the protocol reserve.

Rho is in an experimental beta phase. Only risk funds you are willing to lose! The protocol may be redeployed once the trial period expires.

Swap durations are 7 days, cDAI borrow rates are used as a benchmark, and cDAI is used as collateral. The beta deployment includes a liquidity limit, which if reached, will pause new swaps and new liquidity provision. The limit will be increased over time. 

Rho is deployed at: [0xEC41a154386fe49aFa1339C5419eCB8f19a61294](https://etherscan.io/address/0xEC41a154386fe49aFa1339C5419eCB8f19a61294)
The app is deployed at: [https://beta.rho.finance](https://beta.rho.finance)

## Opening Swaps 
* `openPayFixedSwap(uint notionalAmount, uint maximumFixedRateMantissa)`
* `openReceiveFixedSwap(uint notionalAmount, uint minFixedRateMantissa)`

Each leg of the swap (floating and fixed) is calculated using a notional principal amount. For example, sending opening a `payFixed` swap with a notional amoount of 100 DAI would mean the user pays 7 days of the fixed rate on 100 DAI, and receives the next 7 days of the floating rate on 100 DAI.

The min and max rate arguments are provided as safeguards against liquidity provider frontrunning attacks. Removing liquidity before a swap is executed will move the offered rate. If the rate at tx execution exceeds the given rate limit, the transaction will fail. Currently, 10% bounds are set by default in the app. 

## Closing Swaps 
* `function close(bool userPayingFixed, uint benchmarkIndexInit, uint initBlock, uint swapFixedRateMantissa, uint notionalAmount, uint userCollateralCTokens, address owner)`

After the minimum swap duration has elapsed (set to 7 days in beta), any address can close any other user's swap for them. Cash flows continue to accrue until the swap is closed. 
After the grace period (set to 12 hrs in beta) has elapsed, any user closing a swap will earn a piece of the swap's equity. The amount increases the longer the swap is left open. Incentivizing swaps to be closed helps protect liquidity providers. 

## Providing liquidity 
* `function supply(uint cTokenSupplyAmount)`
* `function remove(uint removeCTokenAmount)`

Liquidity providers effectively take the other side of every user's swap. They earn the spread between the fixed rates they receive and the ones they pay. Unless there are equal amounts of either type of swap, the LPs *will* have exposure to the floating benchmark rate. This exposure could be hedged by opening swaps. 

## Admin 
* `function _changeAdmin(address admin_)`
* `function _delegateComp(address delegatee)`
* `function _transferComp(address dest, uint amount)`
* `function _pause(bool isPaused_)`
* `function _setLiquidityLimit(uint limit_)`
* `function _setCollateralRequirements(uint minFloatRateMantissa_, uint maxFloatRateMantissa_)`
* `function _setInterestRateModel(address newModel)`

The protocol has admin functions to allow parameterization. The admin does not have the ability to impact ongoing swaps, though it does have the ability to change the interest rate model and collateral limits for future swaps. If used maliciously, these actions could impact liquidity providers. 

## Interest Rate Model

Rho's interest rate model quotes more slippage if there is less liquidity is in the pool. The spread between the rate it offers to pay and receive increases as the protocol's utilization increases.

## Audit 

ABDK Consulting performed a brief (2 person weeks) audit on commit [24c166155a70db038aee67740b44cbac3646a365](https://github.com/Rho-protocol/rho-contracts/releases/tag/v0.2-alpha). You can find the annotated report [here](https://docs.google.com/document/d/1HAw1EjPbLUGEOBLq5tq01w3zGeA8fK5IiRjcRfI9S6E/edit?usp=sharing).

## About

I am maintaining Rho on a part-time basis on top of my full-time role at Compound. If you would like to contribute to the protocol or have ideas, please reach out on [Discord](https://discord.com/invite/Pvhn5fTVsm). I'm doing my best to keep development open and transparent, and involve the community. 

## Additional Resources
* [scenarios](https://docs.google.com/spreadsheets/d/1w2EEdeKWvx7haG0p8vp5h9kBmOGBXVOpb6UTZOOV1io/edit#gid=537619964)
* [model](https://observablehq.com/d/d04daaa430a6de46)
* [whitepaper](http://maxcwolff.com/rho.pdf)
