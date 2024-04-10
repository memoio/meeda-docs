# What's Meeda

[Memolabs](https://memolabs.org/) has launched an efficient and simple Ethereum DA solution, which provides reliable data availability guarantee for execution layers such as Layer2 Rollup. Different levels of availability guarantees are provided based on different cost options.

This solution is named Meeda, which extracts the letters from MEMO, Ethereum, and data availability, and is linked with the Bitcoin data availability solution Mooda launched by Memolabs, the first two letters together form MEMO.

## DA Problem

For blockchain, data availability (DA) is a very important thing. As we all know, consensus is an important part of the blockchain, and if nodes want to reach consensus, they must ensure reliable data availability. Nodes can access the transaction data of the current block to verify whether the transaction is correct, thereby preventing malicious transactions from being uploaded to the chain, maintaining the correct "accounting" record of the blockchain, and ensuring the security and reliability of the blockchain.

However, in an overall blockchain like Ethereum, DA is usually used as part of a single system design. When the block space is limited and the block space utilization is high, the gas fee will become higher and higher, and the cost of a single transaction will become higher and higher. The higher the value, the worse the user experience will be and the development of Web3 will be restricted.

Ethereum has also become aware of the expansion problem in recent years and has begun to explore various off-chain expansion solutions. Currently, Rollups have become a popular solution. However, when the Web3 ecosystem is prosperous and the demand for block space is high, it is still facing the problem of excessive gas fees.

## Current DA Solutions

Based on the current situation, there are currently two main solutions. The first is to optimize the storage overhead of Layer 2, such as Ethereum’s Danksharding proposal, which currently uses EIP-4844 to implement blob transactions and reduce the storage overhead of Layer 2 when relying on Ethereum as the data availability layer; the second is modular blockchain solutions, design the execution layer, consensus layer, and data availability layer separately, such as the Celestia project, which aims to simplify blockchain deployment by providing a pluggable consensus network.

<img src="../../images/now-resolve-method.png" title="" alt="" data-align="center">

## Meeda Solution

### Meeda Overview

Currently, Layer 2, as an expansion solution for Ethereum, can safely and reliably expand Layer 1, improve throughput and reduce transaction fees. But there is still a problem: it cannot reduce data synchronization overhead and storage overhead. The original transaction data still needs to be submitted to the L1 chain, and the full nodes on L1 still need to synchronize these transaction data. Even though there are some other solutions, such as ERC-4337 to compress some transaction data, and EIP-4844 introducing the `blob` transaction type, the effect is still limited. The most intuitive manifestation is that storing these transaction data on the L1 chain still requires high gas fees, although these fees are byte fees rather than execution fees.

The fundamental reason is that Layer2 regards the Ethereum L1 chain as its own DA layer (Data Availability). Meeda stores data off-chain, and stores the index used to obtain the data and the proof of commitment to ensure data reliability on-chain. While ensuring data availability, it also reduces synchronization overhead and storage overhead on the chain, maximizing the expansion of the blockchain. Ensuring data security relies on the corresponding challenge and verification mechanism of the Meeda layer. Therefore, Meeda's access is also very simple and efficient, and can be compatible with any Layer 2 chain.

<img src="../../images/structure.png" title="" alt="" data-align="center">

Meeda can be used as a data availability solution for Layer 2, providing low-cost, highly scalable data availability guarantee for Layer 2, and forming a reliable triangular relationship with blockchains such as Ethereum. You can also cooperate with modular solutions to participate in modular blockchains as a data availability layer, providing developers and users with a simpler and more convenient way to build and use Web3.

Off-chain, nodes that issue transactions upload transaction information to Meeda. Meeda ensures the availability of transaction information, and nodes can quickly and easily read transaction data. On the chain, Meeda needs to regularly submit data availability proofs to the chain. If the proof is passed, it can be considered that the availability of user data is guaranteed. If the certificate fails, data repair will be triggered. Among them, the verification method of file availability proof is efficient and low-cost. It adopts optimistic multi-round interactive verification method, KZG polynomial commitment and other technologies to achieve this feature; Meeda conducts availability spot checks on transaction data, and configures a flexible scheme of erasure codes and multiple backups, thus greatly improving data availability.

### Meeda Architecture

Meeda's own logic is divided into two parts: off-chain and on-chain. The off-chain part is called the storage layer, and the on-chain part is called the DA layer. By storing the data off-chain, and storing the index used to obtain the data and the proof of commitment to ensure data reliability on the chain, it not only ensures data availability, but also reduces the synchronization and storage overhead on the chain. Overhead, maximally scaling the blockchain.

Specifically, the off-chain part is responsible for storing transaction data, providing data reading functions, responding to data availability proof challenges, and performing data repair. The on-chain part is responsible for ensuring data availability. It mainly records the commitment value and metadata information of transaction data, generates storage challenge information, records the data availability proof submitted under the chain, responds to the availability fraud proof, and incentivizes data repair.

<img src="../../images/da-structure.png" title="" alt="" data-align="center">

### Meeda's Data Availability

Meeda has continuity proof and on-chain verification, and Meeda's sampling verification does not require downloading data, and users do not need to run verification nodes. Meeda has a reliable, trustworthy, and concise verification method, and strives to minimize costs. In addition, Meeda can also support the existing light client verification method in the Ethereum community, but Meeda chooses more efficient and concise on-chain verification.

Meeda's sampling verification is an adaptive verification method. It performs full challenges on new blocks (e.g. blocks in the last 10 epochs) and sample challenges on old blocks. DA needs to ensure that each block is available. Therefore, for blocks containing transactions that have not been finalized, Meeda will conduct a full challenge to fully ensure the availability of data. For older data, Meeda uses a sampling challenge. We assume that the data failure rate is 1/1000, which means that only 1 in 1000 files is faulty (for storage, this level of evil is not profitable. Assuming 100GB of data, the evil node only stores 99.9GB data, so for him, the risk is greater than the benefit, so the failure rate of 1/1000 is almost the lowest failure rate), then the probability of not finding the wrong data after challenging m times is (1-1/1000)^m, then the fault data is found to be 1-(1-1/1000)^m. We assume that the fault discovery rate is 99.99%, which means that the fault can be found with 99.99% certainty. Then we can get m to be approximately equal to 9205.7. We assume that each sampling challenge is 10,000 blocks, so that the probability of discovery failure data is greater than 99.99%. This adaptive sampling check enables us to 100% guarantee the data availability of recent blocks, and guarantee the data availability of historical blocks with a probability of greater than 99.99%.
