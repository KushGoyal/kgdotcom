---
layout: post
date: 2021-08-31
title: An detailed explanation of Ethereum Layer 2 Solution Polygon Matic Network [DRAFT]
categories: [crypto, coding]
image: /images/2021-08-04/calls.jpg
---

Matic is a [sidechain](https://blockstream.com/sidechains.pdf) which uses an adapted version of [plasma MVP](https://www.learnplasma.org/en/learn/mvp.html#plasma-mvp) to secure itself using the Ethereum mainchain.

Sidechain and plasma chain are 2 different concepts. Matic uses both the concepts to create a secure and scalable blockchain which is interoperable with the Ethereum main chain.

Sidechain is another blockchain which can interact with the “parent” chain using a bridge. Sidechain is essentially a completely independent blockchain which uses pegging to enable interoperability.

Plasma chain is another blockchain which posts root of each block to the parent chain. This is done to draw security from the a more secure blockchain. Once the block hash is store on the main chain it is considered final. The posting of the root in the parent chain is called checkpointing.

<!--more-->

To create a checkpoint blocks produced between the last checkpoint are merklized and the Merkle root is published on the ethereum blockchain.

If the sidechain is compromised the user cannot prove that they own tokens on the sidechain since there is no proof of the ownership on the parent chain. In case of plasma using the regularly posted checkpoints the user can prove that they own the tokens on the plasma chain by showing the existence of the transaction to own the tokens via checkpoint.

Validator nodes are responsible for validating blocks and checkpointing the blocks on the Ethereum mainchain.

Checkpoints are important for two reasons:
1. Providing finality on the Root Chain
2. Providing proof of burn in withdrawal of assets

A checkpoint is the Merkle root hash of the blocks between a time interval.

## Architecture
Heimdall: validation and checkpointing layer
Bor: block producing layer

Matic is a PoS based blockchain which uses modified plasma based checkpointing mechanism to draw security from the Ethereum main chain.

## PoS bridge
A bridge uses smart contracts on both polygon and Ethereum to enable token transfer between the 2 blockchains.

When tokens are to be transferred from Ethereum to Matic a user sends ERC tokens to the `Predicate contract` in which the tokens are locked at the Ethereum end. The Matic chain scans the contract to mint the tokens in the Matic chain.

When tokens are transferred back from Matic to Ethereum the user burns the tokens in the Matic chain. The transaction id is submitted to the `RootChainManager contract` on Ethereum which verifies it using the checkpoint to release the locked tokens. This verification of proof of burn using the checkpoint can take from 20mins to 3hrs.  

### Plasma Bridge

Plasma bridge uses UTXOs to represent funds. When you transfer funds from ETH to Matic chain the contract on ETH end creates UTXO to represent the deposited funds. Matic chain reads the funds from the events emitted by the Ethereum smart contract and mints tokens of equivalent value.

After the user has done all the transactions on the Matic chain they can transfer the tokens back to Ethereum. To do this first the tokens on Matic chain are burned using a withdraw transaction. The receipt of this withdraw transaction is summited on Ethereum. The withdraw transaction is mined in a block and the proof of inclusion of the block in a checkpoint is also submitted to Ethereum. The proof of inclusion is needed to verify if the withdraw transaction was really performed on Matic.

The receipt of the withdraw transaction is in the form of a UTXO. UTXOs are verified for 2 conditions:
1. The owner of the UTXOs is the same as the user submitting the withdraw transaction
2. The transaction that created the UTXO is already included in a block

These 2 conditions does not verify if the UTXOs are confirmed to be unspent. Plasma bridge has a 7 day challenge period for withdrawal of ERC tokens from Matic to Ethereum. This period is needed to make sure that the UTXOs submitted in the withdraw transaction are confirmed to be unspent.

## High TPS and Low fees
Polygon is a POS based blockchain which uses Tendermint consensus engine. There can be 100 validators in the network and rest of the users have to delegate their tokens to these validators.
