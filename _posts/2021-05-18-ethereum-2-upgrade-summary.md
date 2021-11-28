---
layout: post
date: 2021-05-18
title: Ethereum 2.0 upgrades explained in 5 minutes
categories: [crypto]
---

Ethereum is facing a scaling issue because of high transaction volume. More than USD 1T were settled on Eth in 2020 and blockchain has reached its peak in terms of transactions per second. Gas fees is very high and doing a transaction on ethereum is no longer accessible for most users because of it.

**Eth 2.0** will solve the scalability issues by a combination of massive upgrades. I will summarise the new upgrades in this post. Eth 2.0 is still in development though and can take any of the numerous directions proposed by the Ethereum research team.

Eth currently uses proof of work (PoW) as a consensus mechanism. Every 12.5s a block is produced and eth mining nodes execute smart contracts, verify the transactions and maintain the state of the blockchain.

<!--more-->

Eth is now transitioning to **proof of stake** (PoS) consensus mechanism which involves miners to stake eth and run a validator node. Eth 2.0 upgrade is divided into several stages and at the end of the upgrade transition to PoS will be complete along with other upgrades to increase the throughput of the blockchain.

Eth 2.0 is divided into several phases. Phase 0 is the launch of the beacon chain and it is already live. **Beacon chain** is the base layer of the eth2.0 chain. It is the chain on which everything gets finalised. At the moment current eth PoW blockchain and beacon chain are running in parallel.

Beacon chain uses PoS to finalise blocks.  Miners have been replaced with validators who now validate blocks instead of mining them. Each validator  has to stake 32 eth.  A single node can run multiple validators. 

Eth PoS uses an algorithm called **RANDAO** to randomly select proposers who propose the block and validators who vote on the validity of block.  If a proposer submits an invalid block which does not get majority vote they get penalised and their staked eth gets slashed. Validator are also punished via slashing if they vote for a block which does not get included in the chain. And for an attack one third of all validators need to vote for invalid blocks. But this is super expensive since the slashing mechanism will make the cost of maintaining such an attack infeasible.

Currently there is no activity on the beacon chain except distribution of staking rewards. Staking is done via depositing eth1 to a smart contract on eth1 PoW blockchain which then connects with the beacon chain somehow to register the transaction for staking the same amount of eth2.

The next stage of eth upgrade is the merge or docking. This will convert the eth1 PoW blockchain into a eth2 PoS shard. Beacon chain will have 64 shards and the eth1 chain will be one of those shards. Each shard will be able to store data and execute smart contracts which will scale up the throughput by 64x compared to a single chain.

The **validators** will validate blocks of shards and then these validated blocks will be cross linked to the beacon chain. The process of cross linking is required to confirm the shard blocks and subsequently the transactions in the blocks. Each beacon chain block can contain 0 to 64 crosslinks. When a cross link is included in a beacon chain block it is not considered confirmed until another beacon chain block is added. Similar to how blocks are considered confirmed in PoW blockchains.

The merge phase is a mega huge milestone since eth blockchain will completely move to PoS. Making it eco friendly since no random computation is needed to generate a nonce. 

PoS will reduce the eth issuance rate compared to PoW. Another upgrade which will affect issuance is **fee burning**. Currently the miner who produces the block takes all the fees. After the hard fork to introduce fee burning, miners will only get a tip or preference fee and the actual fee to process the transaction will get burned. 

In PoW the miner who finds the block collects both the block reward and the fee from the transactions in the block. In PoS there is one block proposer and several validators who vote for the validity of the block. Proposer and validators get rewards if the block gets included in the blockchain. With fee burning the block proposer will be able to collect only the tips specified by the users. The tip is an incentive for faster inclusion in the block.

The merging will be done in a way to keep the history of the blockchain intact and use the same virtual machine for contract execution. So that existing contracts are not disturbed.

The next phase will be release of **shard chains**. Each shard is identical in functionality. Perhaps the eth1 shard is different. This is not clear currently if all shards will be able to execute smart contracts. Based on the eth website all shards will be able to execute code. But it is also proposed that shards will only store the state of the transactions and not execute code. Code execution will move to layer 2 solutions which will make eth2 much scalable. However eth1 shard will retain its ability to execute code. All this is too confusing and unclear for now. If layer 2 solution adaption comes earlier than the release of this phase then perhaps shard chains might not need to execute code at all. This is yet to be seen though.

**Layer 2** solutions are already in the market and many more are coming. There are several different approaches to layer 2 solutions. Optimistic rollups, Plasma chain, Zk-rollups and State channels. My personal favourite are rollups since they batch several transactions together and perform computation off chain.

**Optimistic rollups** execute smart contracts off-chain via sequencers. Each sequencer will publish the state of the transactions on-chain. These states can be verified by any verifier for validity. If it is found to be invalid the sequencer loses eth in their bond which was needed to become a sequencer. The incentive to become a sequencer is to get the withdrawal fee from users. This is because users cannot withdraw funds right away, there is a dispute period of some days during which the transaction can be disputed. The sequencer can however provide withdrawals from their own pocket by charging a fee. A sequencer is confident that no fraud has been committed and can collect the locked eth after dispute period is over.

**Plasma chains** are layer 2 blockchains which use eth layer 1 to provide additional security using fraud proofs. If any malicious transaction happens on the plasma chain it can be disputed on the eth layer 1 using fraud proofs. Plasma chains keep posting states to layer 1.  Plasma chain is great for scalability since it manages all the complexity but draws security from layer 1.

Eth2.0 is a colossal upgrade and will change eth landscape. Most of it is just about to get live in 2021. 3 main takeaways are: PoS, low issuance rate and layer 2 solutions.

### Read more
<https://research.paradigm.xyz/rollups>  
<https://ethos.dev/beacon-chain/>  
<https://ethereum.org/en/eth2/>  
<https://ethereum.org/en/developers/docs/scaling/>  
<https://docs.ethhub.io/ethereum-roadmap/layer-2-scaling/plasma/>  
