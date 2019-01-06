# Introducing Ignis: Fire Plasma

### SNARK-driven Plasma with up to 500 tx/sec goes live on testnet

![](https://i.imgur.com/MjTQo7I.png)

:::info
*If you're a Plasma developer, feel free to skip the introduction and scroll down right to the tech details.*
:::

Launch of Bitcoin network [exactly 10 years ago](https://www.blockchain.com/btc/block/000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f) opened a new era in technology, politics and economics. Satoshi's invention lead to the creation of a mind-blowing $100+ billion-dollar industry and a huge paradigm shift in people’s minds: The Revolution of Decentralization has begun.

It was also a great reminder that the progress of human race is shaped much sharper by Black Swans and innovative breakthroughs than by incremental development. A decade of blockchain history has proven this point over and over again. Ethereum is one awesome example of a zero-to-one leap. Yet, oftentimes things missing speak much more eloquently than things present. Diverse aspects of cryptocurrencies and blockchain technology have improved over time, but one thing has barely changed much despite lots of effort: scaling.

We need a little clarification here. Many projects claim to have solved the scaling problem. It is indeed trivial to do so: a single Postgres database can handle millions of transactions easily. The real challenge is: scaling without compromising decentralization and security. This is much, much trickier.

Bitcoin is capable of 5 transactions per second, Ethereum manages just 15. In contrast, Visa averages 2000 TPS, with peaks reaching tens of thousands TPS. Take a moment and think about it: how would the blockchain space look if distributed ledgers could maintain the parity? If big industries and institutional investors expected payments and contractual operations on blockchain to be smooth and reliable at any volume, and not be easily disrupted by something like a cute kitty video game? Where would we be now?

A breakthrough is long awaited.

### Plasma Recap

One promising idea of scaling was [proposed by Joseph Poon and Vitalik Buterin](https://cointelegraph.com/news/vitalik-buterin-bitcoins-lightning-network-author-reveal-ethereum-scaling-plans-analysis) in Summer 2017 under the name Plasma. The main idea was to eliminate unnecessary data in smart contracts and only broadcast short cryptographic commitments to blocks on the public mainchain. Verification of correctness of the committed transactions and storing the transaction data was supposed to be happening offchain. Users noticing incorrect data being committed can challenge the operator in EVM and revert the block.

The latter aspect happened to bring a lot of technical challenges with it, gradually discovered by teams implementing Plasma over the last year. The biggest problem is that users are forced to constantly monitor at online activity. This means that: 1) enormous data volumes must be transferred and stored offchain, and 2) that users without gapless online presence can not  security guarantees comparable to that of the mainchain.

### A New Hope: Zero Knowledge Proofs

Inspiration for our work came from a technology which has seen dramatic progress in the last couple of years: [Zero-Knowledge Proofs](https://en.wikipedia.org/wiki/Zero-knowledge_proof) (ZKP). It’s a family of ingenious mathematical techniques, which, among other properties, provide proofs of computational integrity -- in other words, they allow us to convince an external observer that certain computations have been performed correctly, and to do so in a succinct and easily verifiable way. For introduction into ZKP see the [awesome zero knowledge list](https://github.com/gluk64/awesome-zero-knowledge-proofs) on github or Vitalik's [brilliant tech blog series](https://medium.com/@VitalikButerin/zk-snarks-under-the-hood-b33151a013f6).

Specifically, we have chosen a ZK-technique called SNARKs, based on the [Groth protocol](https://eprint.iacr.org/2016/260.pdf). SNARKs are an especially good fit for our problem for a few reasons:

- They are well studied and understood by cryptographers.
- Verification complexity is O(1), which is currently unbeatable. It means that verifying proofs onchain will remain constant (and cheap) regardless of the number of transactions processed.
- Ethereum has precompiled contracts for the math primitives required to implement SNARKs since the Byzantium hard fork.

The biggest challenge to using SNARKs is the requirement of a Trusted Setup, however this problem is solvable. We will write more about it in a separate post.

## Enter Ignis: SNARK-driven Plasma

Today, we’re happy to present the first ever fully functional **alpha version of a SNARKs-driven Plasma**, launched live on Rinkeby testnet. We called it *Ignis* -- Fire.

**<a href="https://ignis.thematter.io/" target="_blank">CLICK FOR LIVE DEMO</a>**

In Ignis verification of transactions by users is replaced by the following approach: operator(s) proposing blocks must submit a SNARK proving that the new block is correct, which is verified automatically by the smart contract. No incorrect block can ever be included by an operator, so users do not need to always be online and constantly monitor transaction activity.

Unfortunately, data availability remains an open challenge which does not seem possible to solve without trade-offs. Ignis follows the onchain data approach [proposed by Vitalik Buterin](https://ethresear.ch/t/on-chain-scaling-to-potentially-500-tx-sec-through-mass-tx-validation/3477). A small piece of data (9 bytes in our case) from every transaction is posted to EVM, in order to guarantee that everybody can reconstruct the Merkle state. 

Technically, this violates the narrow meaning of the term Plasma, because the cost of storing data remains linear to the number of transactions. However, since we achieve a gas cost reduction by a factor of 50 compared to normal token transfers, we believe that using the name Plasma is justified, as the resulting architecture resembles the original construct and achieves its declared goals to a high extent.

## Technical Description and Design Philosophy

Under the hood, *Ignis* is a [L2 solution](https://ethereum.stackexchange.com/questions/47229/what-exactly-is-ethereums-layer-2) with **account model** and **verifiable** state transitions. What we call *state* is the set of current balances of all the accounts, characterized by a root hash. Verifiable state transition means that correctness of every transaction included in the next block must be proven by SNARKs.

E.g. for a transfer transaction between accounts, following checks are performed:

- Sender had enough funds to make a transfer.
- Sender has provided a valid signature.
- The transaction is well-formed, has proper nonce and validity date (should be included not later than a block N).
- Sender's balance was properly reduced and a nonce increased.
- Recipient's balance was properly increased, and no other information about the recipient's account was changed.
- If there was some fee paid for this transaction - this fee was correctly added to the total fee collected over this block.

Similar protocol is followed for other types of transactions.

### Workflow From The User's Perspective

Users submit transactions to one or more operators, who collect them in blocks and submit these to the mainchain along with the SNARK proofs.

User's interaction with *Ignis* can take form in four different operations (transaction types):

- **Registration/Deposit**: users can register their public key and obtain a free account number in *Ignis*.
- **Transfer**: users can either send a part of his funds to another user.
- **Partial Exit**: or can send them to the special address of zero. Transfer to zero is considered as a partial exit (different from complete exit described below), so as soon as a block with such transaction is verified, the user can withdraw a corresponding amount using the smart-contract in Ethereum network.
- **Full Exit**: this is what we call a "complete exit". Such operation withdraws a complete user's balance and un-registers his public key.
- **Emergency Exit**: in case *Ignis* network experiences a long-term downtime (all participants are not processing transactions), smart-contract goes in "emergency mode" and allows every user to exit full balances using smart-contract.

Deposits and Exits are initiated by the use directly on smart contract via Ethereum transactions, transfers are submitted offchain (e.g. by a peer-to-peer network).

## Decentralization and Censorship-Resistence

We envision *Ignis* in the future as a decentralized network in multi-operator paradigm from day one. At the current stage we focused on highlighting the SNARKs-related workflow and primitive operations in the live demo, while the multi-operator part is a work in progress.

### Multi-Operator Model

To support efficient multi-operator model, we propose a delegated Proof-of-Stake to assemble a pool of M operators. A random lottery with chances proportional to the stake elects an operator with the right to produce blocks during the next time window. If an operator elected to provide blocks in the current time window experiences technical difficulties with proof generation, a smart-contract may allow the next operator to step in.

PoS is required for the following major reasons:

- To separate the basic operator functions (collecting and including transactions in blocks) from the actual proof generation. This way we can prevent concentration of too much power in the hands of a single operator with the most efficient proving hardware and thus avoid censorship.
- To prevent deliberate or occasional denial-of-service by operators that cease to produce blocks. Stake will serve a role of security deposit which is burned if an operator doesn't deliver their promises.

To make multi-operator model efficient we suggest a 2-phase commit-verify approach: blocks are first committed on chain, their verification with SNARK proof follows. Such a scheme enables signaling of the content of the next block to other operators. This allows operators to perform heavy SNARK-proof computations in parallel, thus making latency (time from submitting a transaction to its finalization on chain) and throughput (number of transactions per second) independent of each other. See the picture below for illustration:

![Commit-Verify](https://i.imgur.com/fCBqdXA.jpg)

## Benchmarks and Limitations

### Througput

Throughput is the number of transactions per second (TPS) a system can handle.

Theoretical limit of throughput in Ignis is set by its onchain data availability approach: since some data has to be posted onchain for every transaction, an Ethereum block can accomomdate not more than ~500 TPS. In practice gas cost will likely go wild while approach this limit, but we believe that a few hundred TPS is realistic.

There is an ongoing research about alternative, offchain data availability approaches. If it is successful, the limiting factor shifts to the number of transacations which can be packed into a block multiplied by number of SNARK proofs that can be verified in EVM per second.

Currently EVM only supports native primitives for BN256 elliptic curve pairings. This curve field has $2^{28}$ cubic roots of unity, which means that we can efficiently verify SNARKs up to ~256 Million constraints. We managed to successfully pack and verify 1600 transactions under these limitations (the computation took 20 minutes on a 72-core AWS server). With some tricks and optimization this will allow us to verify up to 3500 TPS on EVM.

### Latency and Tx Cost

Transaction costs in Ignis consist of 2 parts: onchain gas costs and costs of offchain proof generation (the latter is independent of the Ethereum gas price).

Ignis currently requires a fixed 1K gas per tx onchain.

The offchain cost and latency (time to include a single transaction from the moment of broadcast till final block verification) are interdependent in Ignis. The lower the latency, the more power server is required to compute proofs, thus the cost per tx is higher.

At the target latency of 5 min at 100 TPS we estimate the offchain part to be approximately 0.001 USD. This estimate is very conservative, further optimizations can push it down.

## Next Steps

- Move from the "proof of concept" to the production-grade software that would allow anyone to become an operator with ten lines of configuration.
- Push the efficiency of the proof-generation software further.
- Complete the economic model of delegated PoS for the multi-operator approach.
- Conduct security audits.
- Do the trusted setup! It's an essential part for any SNARKs based solution and the main blocker for the production release on the Ethereum mainnet.

## Buidl and Contribute

- All our work on SNARKs is public on the [github](https://github.com/matterinc).
- Code examples can be found in our ETH Singapore repository - [Plasma Winter](https://github.com/matterinc/plasma-winter) and [Plasma Cash history compression](https://github.com/matterinc/plasma_cash_history_snark).
- Complete code of Ignis Alpha will be published after a clean-up in short -- stay tuned!

## Progress Tracker

Current status of the project can be tracked [here](https://github.com/matterinc/ignis/blob/master/progress/README.md).

## Credits

- [ZCash](https://z.cash) for their various contributions to SNARKs environment.
- [Sean Bowe](https://twitter.com/ebfull) for his [awesome code](https://github.com/zkcrypto/bellman).
- [Vitalik Buterin](https://vitalik.ca) for his educational blog posts and ideas on [ethresear.ch](https://ethresear.ch).
- [Karl Floersch](https://karl.tech) for his energy!
- [BarryWhitehat](https://github.com/barryWhiteHat) and [HarryR](https://github.com/HarryR) for the great work on [rollup](https://github.com/barryWhiteHat/roll_up) and integrating SNARKs into Ethereum.
- [Ethereum Foundation](https://www.ethereum.org/foundation) and personally Albert Ni for the support of community-driven R&D projects.

## Contact

Comments and questions are highly welcome! Please [contact us on gitter](https://gitter.im/ignis-plasma/community#).

[Alex Vlasov](https://github.com/shamatar), [Matter Inc.](https://thematter.io/) 

[Alex Gluchowski](https://twitter.com/gluk64), [Ethereum Foundation](https://www.ethereum.org)
