---
layout: post
date: 2021-06-27
title: Cosmos Network Explained - Cosmos Hub, ATOM tokens, Gravity DEX
categories: [crypto]
image: /images/2021-06-27/cosmos-zones-4k.png
---
[Cosmos](https://v1.cosmos.network/resources/whitepaper) is a network of interoperable blockchains. Cosmos uses [Tendermint](https://docs.tendermint.com/master/) BFT which is PoS based consensus algorithm. Tendermint BFT is an open source platform and can be used by any project.

To create a network of blockchains Cosmos provides [CosmosSDK](https://docs.cosmos.network) (the application layer) and Tendermint SDK (the network and consensus layer) to give complete tools to launch a public or private blockchain. All the blockchains will implement [IBCI](https://ibcprotocol.org) (inter blockchain communication interface) so that they can transfer tokens and data between each other seamlessly.
<!--more-->

## Horizontal Scalability
The idea behind [Cosmos network](https://v1.cosmos.network/intro) is to have horizontal scalability. Blockchains are distributed databases and need globally distributed nodes to store and maintain the state of the database. Everyone is allowed to write to the database in a secure way provided they follow the rules of the protocol. The consensus rules secures and maintain the globally accepted state of the blockchain but come at the cost of scalability. 

Bitcoin solves consensus and security by using Proof of work (PoW ) consensus which puts a hard cap of 10 minutes per block. This limit along with probabilistic finality makes the Bitcoin transaction confirmation time to be around 60 minutes considering a 6 blocks confirmation. Finality is probabilistic because PoW follows the longest chain. To determine the longest chain you have to wait for the chain you are following to be longer than other competing chains.

[Proof of stake](https://ethereum.org/en/developers/docs/consensus-mechanisms/#proof-of-stake) (PoS) is another consensus algorithm which is much more energy efficient than PoW. Ethereum is now migrating from PoW to PoS. Many PoS chains are already live including CosmosHub. PoS does not solve scalability of blockchains. PoS needs validators to validate blocks and broadcast their vote for validity of the new block to the network. Communication between nodes is limited by physical laws (speed of transmission, packet size) which brings latency. To validate each block each validator communicates with the whole network so latency is proportional to nC2 where N is the number of validators.

On top of the network latency there is the time and cost of validating each transaction. Validating smart contract transactions can be expensive since each node has to run the whole code execution to determine the final state of the transaction. When targeting high TPS this can be a bottle neck because it will require nodes to have very good CPU configuration and low network latency.

To support high TPS in a PoS network you will have to compromise on decentralisation to reduce network latency. Tendermint limits the number of validator nodes to a specific number which can be set by the users of the network. By limiting the number of validators the total number of network communications between validators gets fixed.

Ethereum 2.0 (PoS based) is solving scalability by using using 64 shard chains. Sharding is a known method to horizontally scale the database. The block time in Ethereum will be 12s after migrating to PoS which is just .5s faster than the current block time of 12.5s. With 64 shards the transaction speed of Ethereum will be around 1000 TPS. With a variety of layer 2 solutions Ethereum aims to bring transaction speed to 100K TPS.

To support the global financial system it will take a lot more than 100K TPS. Cosmos network does not want to be limited by the TPS of a single blockchain. Cosmos network will have many blockchains which can communicate with each other. Each blockchain can use layer 2 solutions to scale itself. With no limit on number of blockchains connected to the network, scalability can be solved by adding more blockchains when existing ones are full up to the maximum TPS. 

Cosmos network will have blockchains for [specific use cases](https://blog.cosmos.network/why-application-specific-blockchains-make-sense-32f2073bfb37) which makes the network more efficient since blockchains can choose to not add a virtual machine in the nodes. Application specific blockchains can code the logic in the transaction itself rather than executing code in a virtual machine loaded inside the node. A virtual machine like the [Ethereum virtual machine](https://ethereum.org/en/developers/docs/evm/) is Turing complete and can execute code line by line.

Cosmos network uses the hub and spoke architecture. The spokes are Zone blockchains which will connect to Hubs. Hubs will be used for inter blockchain communication, shared security and bridges to networks outside Cosmos ecosystem.

##  Sovereign ecosystem
Cosmos network has value only when there are other zone blockchains which have implemented IBCI. Cosmos has very loose coupling with the zones. Each zone is an independent blockchain with its own validators and governance.

Using the [CosmosSDK](https://docs.cosmos.network) and Tendermint anyone can create a blockchain from scratch with its own set of validators. Cosmos network team has created a toolkit called [Starport](https://cosmos.network/starport/)  to create a new blockchain with a lot of boilerplate already created for you.

Cosmos has multi token fee mechanism which makes the user experience so much better and opens up the ecosystem. The validators can choose which tokens are eligible to be used as transaction fee. Validators having the most skin in the game will choose what is best for the users.

More sovereignty is a big step towards decentralisation. Any zone can choose not to participate in the CosmosHub IBC anytime.

Popular [projects](https://cosmos.network/ecosystem/tokens) built using Cosmos SDK
- [Terra.money](https://www.terra.money)
- [ThorChain](https://thorchain.org)
- [Band protocol](https://bandprotocol.com)
- [Secret network](https://scrt.network)
- [Oasis network](https://oasisprotocol.org)
- Binance Chain

## Cosmos Hub
A Hub in Cosmos network is used to connect zone blockchains using [IBCI](https://ibcprotocol.org).  Each hub is a blockchain itself and is independent from other hubs. A hub can add any feature as it seems fit for the benefit of users and the value of the native token of the hub.

[Gia](https://hub.cosmos.network/main/hub-overview/overview.html) is the first Cosmos Hub. ATOM is the native token of the Gia hub. Interchain Cosmos team has released a roadmap for [Gia version 2.0](https://github.com/cosmos/gaia/blob/main/docs/roadmap/cosmos-hub-roadmap-2.0.md) which introduces many amazing features.

Main [features](https://cosmos.network/features/) of the Gia Cosmos Hub
1. [Interchain token transfers](https://medium.com/chainapsis/why-interchain-accounts-change-everything-for-cosmos-interoperability-59c19032bf11) (implemented using IBCI)
2. [Gravity DEX](https://gravitydex.io/about/) (planned)
3. [Gravity bridge](https://blog.althea.net/gravity-bridge/) - a bridge to the Ethereum blockchain (planned)
4. [Interchain staking](https://github.com/informalsystems/cross-chain-validation/blob/2b7e5ddebd10c9eea743989ca3fbcba990db4987/spec/valset-update-protocol.md) (planned)
5. [Token issuance](https://github.com/cosmos/gaia/blob/main/docs/roadmap/cosmos-hub-roadmap-2.0.md#lambda-upgrade) (planned)

The Hub is central to the cosmos network. Cosmos Hub is responsible for connecting all the zone blockchains. A Hub is described as a [port city](https://blog.cosmos.network/the-cosmos-hub-is-a-port-city-5b7f2d28debf) from where the users can discover other zone blockchains and also access them.

There can be many hubs in the whole network. Each hub will be connected to other hubs. A zone1 blockchain connected to hub1 can transfer tokens to a zone2 blockchain connected to hub2. 

```
zone1 --> hub1 --> hub2 --> zone2
```

Using IBCI users can manage tokens and do transactions using just their Cosmos Hub address and wallet. A single [interchain account](https://medium.com/chainapsis/why-interchain-accounts-change-everything-for-cosmos-interoperability-59c19032bf11) to operate the whole network of blockchains will be a game changer.

### ATOM
Atom is the native token of the Gia Cosmos Hub. The utility of the Atom is that it is necessary for staking and thus gives you the ability to earn the transaction fees and the block rewards of chains validated by the Cosmos Hub validators, which includes the Cosmos Hub itself as well as any hosted chains.

1. Staking rewards
2. Governance
3. Interchain staking rewards
4. Transaction fees:
	* Token transfer transaction fee
	* Gravity DEX transaction fee
	* Gravity bridge transaction fee

### Staking and governance
Cosmos Hub is a PoS blockchain. Validators have to stake ATOM tokens to be eligible for validating blocks. Currently there can 125 validators who earn the staking rewards. Other users have to delegate their ATOM tokens to any of the 125 validators to earn rewards.

Cosmos SDK has a governance module. Using the governance module the blockchain can be updated by submitting proposals. The validators and delegators vote on the proposal to approve or reject it. If a delegator does not vote their vote gets delegated to the validator.

### Transaction fee
Cosmos Hub is connected to other blockchains called zones which support IBCI. Using IBCI tokens can be transferred from one zone to another. The fee for this transaction can be paid in zone tokens. This fee goes to the validators.

Cosmos has a transaction fee assigned to each transaction type. Some of the transaction types are; SendTx, BondTx, UnBondTx, IBCBlockCommitTx etc. The list of transaction types will keep growing as more features will be added.

The transaction fee for interchain token transfers can be collected in any whitelisted fee token. Cosmos network validators whitelist the fee tokens via on-chain governance.

Cosmos allows each validator to choose relative value of whitelisted tokens to determine the fee of each transaction. The transactions with higher fees get included in the block first. The current mechanism will be replaced by the global fee ranking in which each validator will vote on the relative value of the tokens. This global list ensures soother and predictable experience when paying fees.

Cosmos Hub plans to enable creation of ERC20 like tokens. There will be gas fee associated with transfers and creation of such tokens.

CosmosHub will have various features like DEX, bridges, interchain token transfer, token issuance etc. All operations will have a transaction fee associated with it which will be earned by the validators.

### Gravity DEX
Cosmos Hub will implement the [Gravity DEX](https://github.com/tendermint/liquidity/blob/develop/doc/LiquidityModuleLightPaper_EN.pdf) for trading of zone tokens. Proposed to use a hybrid model of order execution which uses both AMM and order books.

Batch execution of orders is proposed to avoid front running attacks. Orders will pooled for a certain blocks length to be executed together based on the ask price.

Gravity DEX will use equivalent swap price model (ESPM). In this model the swap price is matched with the open market price. To do this constant product does not hold true for the pool.

#### Types of Fees
1. Pool creation fee - To create a liquidity pool a fee of 100 ATOM is charged to prevent spam. This fee is paid to the Cosmos community fund.
2. Pool withdrawal - Fee is paid in the tokens of the pool as is kept in the pool.
3. Swap Fee - the trading fee is charged for each swap as in kept in the pool
4. Network gas fees - A transaction has a gas fee associated with it which is paid to the validators

Gravity DEX will increase adoption of new blockchains in the cosmos network. Users will be able discover new tokens on the DEX and buy them instantly. 

### Gravity Bridge
[Gravity bridge](https://blog.althea.net/gravity-bridge/) will connect the Ethereum blockchain with the Cosmos Hub. Users will be able to transfer tokens between Cosmos and Ethereum using the bridge.

A smart contract on the Ethereum blockchain is used to interact with the Ethereum blockchain. Transactions on ETH side will be batched together for low gas fee per transaction. Low gas fee per transaction is a [great incentive](https://blog.althea.net/the-gravity-well/) for users to use Defi apps via the gravity bridge. Performing swaps on Gravity DEX for ERC20 tokens will be much cheaper than performing them on ETH native DEX like Uniswap.

### Interchain staking
A zone blockchain can use the validator network of Cosmos Hub to validate its transactions. This is shared security. Validators will get rewards for securing the zone blockchain in the zone tokens.

[Cross-Chain validation protocol](https://github.com/informalsystems/cross-chain-validation/blob/2b7e5ddebd10c9eea743989ca3fbcba990db4987/spec/valset-update-protocol.md) is being developed by the Cosmos Hub team to allow validators to validate transactions of the “baby blockchain”. If the validator approves an invalid transaction on the baby chain their staked ATOM gets slashed on the parent chain.

Interchain staking can help bootstrap new blockchain projects without the need for setting up new validator nodes. Interchain staking is an added benefit to the Cosmos Hub validators, who earn baby blockchain tokens as rewards.