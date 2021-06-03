---
layout: post
date: 2021-06-03
title: Terra blockchain and dApps explained in detail
categories: [crypto]
---

Terra is a PoS blockchain built using the [Cosmos SDK](https://v1.cosmos.network/intro). Cosmos SDK uses [Tendermint BFT](https://docs.tendermint.com/master/introduction/what-is-tendermint.html) consensus mechanism which is an open source platform developed by the Cosmos team. Terra is a stable coin platform which uses Luna the base token for stabilising the price of currencies. Terra aims to bring real world assets, currencies and use cases to decentralised economy.

*This post is for educational purposes only. Not investment advice.*

Luna is the native token in Terra. Total 985M Luna are in circulation out of which almost 32% are staked. The current staking reward is 11.2% APY. (Source: [Terra Station](https://station.terra.money))

<!--more-->

Terra has on chain governance which requires the holder of Luna to submit a proposal by depositing some Luna. A minimum of 512 Luna is needed for the proposal to be valid for voting. Proposal for changing any blockchain parameter is automatically implemented if the proposal is passed. But more complex proposals need the community to get them implemented somehow. Terra core team is the one which is will take the implementation ahead. 

Terra supports [smart contracts](https://docs.terra.money/contracts/#developer-tools) platform which uses [CosmWasm](https://docs.cosmwasm.com) (WebAssembly). Terra smart contacts are written in Rust.

Top dApps:
- [Mirror](https://mirror.finance) synthetic assets
- [Anchor](https://anchorprotocol.com) stable yield lending
- [TerraSwap](https://terraswap.io) DEX

## Stablecoins
Terra has an inbuilt automated market maker which maintains the peg for stablecoins. Terra has validator oracles which report the real world prices of stablecoin vs Luna from exchanges. These prices are used in the market making algorithm.

TerraSDR (SDT) is the default stablecoin hardcoded in Terra. SDT is pegged to [SDR](https://www.investopedia.com/terms/s/sdr.asp). There are many other Stablecoins in Terra like USD, KRW, MNT etc. and others can also be introduced in Terra.

Terra uses [constant product](https://docs.terra.money/dev/spec-market.html#market-making-algorithm) market marking algorithm. The market is initialised with equal pools of Luna and SDT called BasePool. The pools are equal in terms of their worth in SDR. The current size of the BasePool is [625K SDT](https://agora.terra.money/t/proposal-to-decrease-poolrecoveryperiod-to-8hour-4800-block/261).  These are [virtual liquidity pools](https://docs.terra.money/dev/spec-market.html#virtual-liquidity-pools) used for protecting against front running the oracles.

```python
# POOL_SDR = 625000
# assuming price to be 0.5 LUNA per SDR
# POOL_LUNA = 625000 / 0.5
# With the price changes POOL_LUNA will keep changing
CP = POOL_SDR * POOL_LUNA * (Price of LUNA / SDR)
```

The size of the `POOL_LUNA`  has to be updated when there is a price change in LUNA <> SDR as reported by the oracles.

After each swap a parameter called `TerraPoolDelta` is updated which is the deviation of the stablecoin pool from the BasePool size. This parameter is decreased over a period called `PoolRecoveryPeriod`. This adjustment is done by the protocol using [ReplenishPools](https://docs.terra.money/dev/spec-market.html#end-block) method.

When Luna is converted into Terra stablecoin it is captured by the protocol and burned, this is called [seignorage](https://docs.terra.money/dev/spec-market.html#seigniorage) . The burned Luna is generated back into the system by giving some of it to the validator oracles as rewards for voting and rest added to the community pool. Stablecoins worth the value of (Luna - spread fees) are minted. [Spread fee](https://docs.terra.money/dev/spec-market.html#swap-fees) also called swap fee is applied to protect from front-running attacks. Terra has a minimum hardcoded spread fee of [0.5%](https://agora.terra.money/t/proposal-to-decrease-poolrecoveryperiod-to-8hour-4800-block/261).

When Terra stablecoin is converted into Luna, all the offered stablecoins are burned and LUNA worth that value is minted.

A user always has 2 choices, use the open markets to swap Luna with UST or use the protocol’s minting mechanism to mint UST. Same is valid when a user wants Luna for UST. The choice depends on the price difference between the price from minting vs the price from buying. Minting is a protocol level activity and it needs to interact with the blockchain directly. While swapping is trading on an exchange.

UST is one of the stable coins on Terra which is pegged to USD. The price of 1 USD in terms of Luna is reported by oracles.  The oracles has reported a price of 10 USD per LUNA while the rate for minting UST is 11 UST per LUNA. If the market price for 1 UST is 1 USD then it makes sense for the user to mint UST from LUNA and sell it for profit of 1 USD. This is a front running attack. This will cause the price of UST<>USD to drop. Terra uses the virtual liquidity pools to prevent such attacks.

Another scenario is when the rate is 1.1 UST per USD on exchanges but the rate of exchange 10 USD per LUNA matches the minting rate of 10 UST per LUNA. User can buy 1 UST from the exchange and then mint LUNA worth 1 UST. Finally sell on the exchange the minted LUNA for USD to pocket profit. Logically the act of buying UST from the exchange will drive the price of UST up and close the gap.

On every stablecoin transaction on Terra a [stability fee](https://docs.terra.money/dev/spec-auth.html#stability-fee) is charged. This fee is distributed among the validators. It can range from 0.1% to 1% of the transaction amount. Luna transactions are exempt from stability fee.

A lot of parameters and concepts discussed above are changing in the next update [Columbus-5](https://agora.terra.money/t/terra-core-priorities-for-q1-q2-2021/388).

## Mirror Finance
[Mirror](https://mirror.finance) is a successful project launched on Terra. It allows creation of synthetic assets called [mAssets](https://docs.mirror.finance/protocol/mirrored-assets-massets). A synthetic asset is not a derivative, it just mirrors the price of the real world asset. To create an mAsset you have to give collateral called collateralized debt position (CDP) in the form of UST or another mAsset. Currently the collateralization ratio is 150%. For example you can give UST 300 as collateral to mint UST 200 worth of mAssets. The process is very similar to [MakerDao](https://makerdao.com/en/).

Mirror gets the market price of the mAsset from [Band Protocol](https://bandprotocol.com) which is an oracle. The prices are updated every 30s. If the price of the mAsset goes up and the ratio goes below 150% your CDP will get liquidated. To prevent this you have to either deposit more collateral or deposit back the mAsset to bring the ratio above 150%.

Minting an asset on Mirror is a short position against that asset. For example  you decide to mint 1 Apple stock worth UST 200 using UST 300 as collateral. This makes the collateral ratio exactly 150%. If the Apple stock increases in value your CDP will liquidated. So you essentially want the Apple stock to drop in value. You are always short the mAsset you are minting against the collateral you are depositing. Another example is depositing UST to mint Facebook stock. Which makes you short Facebook against USD ( not investment advice).

Mirror maintains trading pools of mAssets. For depositing mAsset in the liquidity pool you get a percentage of the trading fee based on your contribution to the pool. Read more about providing liquidity on mirror [here](https://docs.mirror.finance/protocol/lp-token).

Mirror allows only the [whitelisted](https://docs.mirror.finance/protocol/mirrored-assets-massets#whitelisting) assets to be minted. Currently [1.92B UST](https://terra.mirror.finance) has been collateralised to mint 400M worth of mAssets. Assets minted on Terra can be transferred to Ethereum blockchain using [Terra Shuttle](https://github.com/terra-money/shuttle).

## Anchor
[Anchor](https://docs.anchorprotocol.com) is a savings protocol which gives yields powered by the Terra staking returns. The current APY for Anchor is [17.95%](https://app.anchorprotocol.com/earn).

The lender deposits stablecoins in Anchor to earn a stable yield. The borrowers opens a CDP using bonded assets called [bAssets](https://docs.anchorprotocol.com/protocol/bonded-assets-bassets). 

bAssets are fungible tokens received for staking tokens in a PoS chain. They provide liquidity for staked assets and can be used further in financial applications or even sold in open market. The user holding the bAssets gets the staking rewards. 

[bLuna](https://docs.anchorprotocol.com/protocol/bonded-assets-bassets/bonded-luna-bluna) are the bAssets in the Terra chain. bLuna can be used as CDP in anchor. A user has to delegate Luna from a list of whitelisted validators to be eligible to get bLuna. bLuna rate is adjusted based on slashing events.

```python
bLuna redemtion rate = amount of Luna delegated / amount of bLuna minted
```

The amount of Luna delegated can get reduced because of slashing events. For example you delegate 1 Luna and get 1 bLuna. Because of slashing only .96 Luna remains. When you will undelegate the Luna you will get back .96 Luna instead of 1.

The lenders get the staking rewards from the pool of bLuna collateral. Lenders are given [aTerra](https://docs.anchorprotocol.com/protocol/money-market#anchor-terra-aterra) tokens for the interest accrued and the principal they deposited. aTerra value increases with more stablecoin deposits and interest. aTerra can be redeemed for stablecoins.

```python
aTerra redemtion rate = (total deposits + total interest) / total aTerra
```

At T0 the total deposits were UST 100 and 0 interest. Redemption rate is 1 (100/100). At T1 total deposits are still UST 100 and UST 2 interest has been generated. Total aTerra remain the same as no new deposit has been made. Now redemption rate is 1.2 (100 + 2 / 100). With growing interest value of aTerra increases.

[ANC tokens](https://docs.anchorprotocol.com/protocol/anchor-token-anc) are given to the borrowers proportional to the amount borrowed. This is an additional incentive to borrow since ANC tokens can be staked and used for voting.

10% of the yield generated in the Anchor protocol is used to purchase ANC tokens from the ANC<>UST pair from Terraswap. The purchased tokens are distributed to ANC stakers as rewards. This purchase mechanism gives [value](https://docs.anchorprotocol.com/protocol/anchor-token-anc#value-accrual) to the tokens. Users who provide liquidity for the ANC<>UST pair on TerraSwap are also given ANC as [rewards](https://docs.anchorprotocol.com/protocol/anchor-token-anc#distribution-to-anc-liquidity-providers). The whole ecosystem is designed to give value to ANC tokens and thus creating an incentive for users to hold them.

Anchor users vote to set a target yield called Anchor Rate. ANC token holders can vote for setting the Anchor Rate. While the collateral earns the real yield based on staking rewards.

When the Anchor rate is less than the real yield excess yield is stored in the yield reserve. Also ANC token incentive to borrowers is reduced by 15% every week.

When the Anchor rate is more than the real yield the reserved yield is added to the real yield until it depletes. Also the ANC token incentive is increased by 50% every week. 

Anchor charges [interest](https://docs.anchorprotocol.com/protocol/money-market) on the borrowed stablecoins inspired by the [Compound](https://compound.finance) protocol. There is a dynamic borrow limit which depends on the price of bLuna vs Stablecoin as reported by the oracle. If the borrow limit is reached the CDP can be [liquidated](https://docs.anchorprotocol.com/protocol/loan-liquidation).


