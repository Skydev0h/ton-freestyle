# Unchosen projects

This file contains some interesting project ideas that would be interesting to implement, but were chosen not to be implemented for this contest due to one or another reason.



## CryptoKitties / Puppies / *Pythons* / *Turkeys* / etc.

Building games based on blockchain is an innovative and fun idea, however, the usefulness of such application is highly questioned. Collecting things is very fun, however, as for now, there is no official specification of object tokens (with unique metadata) that can be used, however that is not very critical, because such virtual token can be created inside contract's data. More importantly, if building such application, more work, attention and coding is done in the middleware that is linking user interface with blockchain data, and user interface itself. **Therefore, despite anomalously high potential demand from a customer standpoint, this SC poses relatively low practical usefulness.**



## Atomic swap (inter-blockchain)

Atomic swap should allow to exchange assets in one cryptocurrency for assets in another cryptocurrency. The problem is that it is hard to ensure that on the other side the transaction actually happened (because it is not possible to store any secret information in blockchain, contract cannot store some secret and release it just upon transaction), so it requires centralization, some kind of server, and cannot really be currently decentralized. **Therefore, despite great usefulness, this idea was trashed for this contest.**



## Decentralized exchanges

First of all, this SC can be considered in two ways: DX between two different cryptocurrencies and DX between special currency type in TON and Grams. As for the first point, the problem relies on implementation on Atomic swap (look at previous point). As for the second one, currently, I could not reliably find way to mint custom currencies, and even then, the fact that to mint currency you need specify unused currency ID and that minting can only be done once, the approach is not completely reliable (it is possible to collide with used currency ID, and it is difficult to automatically determine result). **More importantly, currently in testnet minter contract is a simple wallet, so actually, it does not currently work.**



## Gambling (dice rolls, lotteries, etc.)

This is analogic to first point (gaming): despite enormous popularity gambling is kind of not serious usage for blockchain, mostly regulated and even forbidden in non-blockchain world, and if done correctly is unpredictable and controlled by mathematical statistics. **That way usefulness of such SC is really doubtful, but potential demand is nevertheless high because people like chance of winning something.**



## Decentralized Autonomous Organization

This could be a very good application to make, both useful and demanded, but because of it's outstanding complexity to make, and that DAOs are still shadowed for me by the Ethereum DAO incident, **I do not think that I could make a reasonable DAO in two weeks (and to be honest - four days, like always...).**



## Contract-contained tokens (ERC-20 for example)

Implementing smart contract that contains tokens such as ERC-20, which would just contain balances of another addresses is unreasonable because there is inherent support of custom currencies in TON, where the blockchain itself guarantees transfers, and smart contracts (wallets) pay themselves for storage of their balance. The only problem is that this machinery is lacking a key gear - *minter contract*, that would allow creation of such tokens (currencies). **I do not see any reason to implement ERC-20 because when minter will be implemented, it will be completely obsolete due to discrepancies with storage and gas processing fee logic.**

