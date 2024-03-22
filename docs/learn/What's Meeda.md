# What's Meeda

Memolabs has launched an efficient and simple Ethereum DA solution, which provides reliable data availability guarantee for execution layers such as Layer2 Rollup. Different levels of availability guarantees are provided based on different cost options.

&nbsp;

This solution is named Meeda, which extracts the letters from MEMO, Ethereum, and data availability, and is linked with the Bitcoin data availability solution Mooda launched by Memolabs. The first two letters together form MEMO (the underlying decentralized cloud storage system launched by Memolabs).

## DA Problem

For blockchain, data availability (DA) is a very important thing. As we all know, consensus is an important part of the blockchain, and if nodes want to reach consensus, they must ensure reliable data availability. Nodes can access the transaction data of the current block to verify whether the transaction is correct, thereby preventing malicious transactions from being uploaded to the chain, maintaining the correct "accounting" record of the blockchain, and ensuring the security and reliability of the blockchain.

&nbsp;

However, in an overall blockchain like Ethereum, DA is usually used as part of a single system design. When the block space is limited and the block space utilization is high, the gas fee will become higher and higher, and the cost of a single transaction will become higher and higher. The higher the value, the worse the user experience will be and the development of Web3 will be restricted.

&nbsp;

Ethereum has also become aware of the expansion problem in recent years and has begun to explore various off-chain expansion solutions. Currently, Rollups have become a popular solution. However, when the Web3 ecosystem is prosperous and the demand for block space is high, it still facing the problem of excessive gas fees.

## Current DA Solutions

Based on this current situation, there are currently two main solutions. The first is to optimize the storage overhead of Layer 2, such as Ethereum’s Danksharding proposal, which currently uses EIP-4844 to implement blob transactions and reduce the storage overhead of Layer 2 when relying on Ethereum as the data availability layer; the second is modular blockchain solutions, design the execution layer, consensus layer, and data availability layer separately, such as the Celestia project, which aims to simplify blockchain deployment by providing a pluggable consensus network.

<img src="..\..\images/now-resolve-method.png" title="" alt="" data-align="center">

## Meeda Solution

### Meeda Overview

Currently, Layer 2, as an expansion solution for Ethereum, can safely and reliably expand Layer 1, improve throughput and reduce transaction fees. But there is still a problem: it cannot reduce data synchronization overhead and storage overhead. The original transaction data still needs to be submitted to the L1 chain, and the full nodes on L1 still need to synchronize these transaction data. Even though there are some other solutions, such as ERC-4337 to compress some transaction data, and EIP-4844 introducing the `blob` transaction type, the effect is still limited. The most intuitive manifestation is that storing these transaction data on the L1 chain still requires high gas fees, although these fees are byte fees rather than execution fees.

&nbsp;

The fundamental reason is that Layer2 regards the Ethereum L1 chain as its own DA layer (Data Availability). Meeda stores data off-chain, and stores the index used to obtain the data and the proof of commitment to ensure data reliability on-chain. While ensuring data availability, it also reduces synchronization overhead and storage overhead on the chain, maximizing the expansion of the blockchain. Ensuring data security relies on the corresponding challenge and verification mechanism of the Meeda layer. Therefore, Meeda's access is also very simple and efficient, and can be compatible with any Layer 2 chain.

&nbsp;

Off-chain, nodes that issue transactions upload transaction information to Meeda. Meeda ensures the availability of transaction information, and nodes can quickly and easily read transaction data. On the chain, Meeda needs to regularly submit data availability certificates to the chain. If the certificate is passed, it can be considered that the availability of user data is guaranteed. If the certificate fails, data repair will be triggered. Among them, the verification method of file availability proof is efficient and low-cost. It adopts optimistic multi-round interactive verification method, KZG polynomial commitment and other technologies to achieve this feature; Meeda conducts availability spot checks on transaction data, and configures erasure codes and multiple backups flexibly. scheme, thus greatly improving data availability.

### Meeda Architecture

Meeda is divided into two parts: off-chain and on-chain.

&nbsp;
The off-chain part is responsible for storing transaction data, providing data reading functions, responding to data availability proof challenges, and performing data repair.

&nbsp;
The on-chain part is responsible for ensuring data availability. It mainly records the commitment value and metadata information of transaction data, generates storage challenge information, records the data availability certificate submitted under the chain, responds to the availability fraud certificate, and incentivizes data repair.

<img src="..\..\images/da-structure.png" title="" alt="" data-align="center">

## Meeda detailed description

### Upload Transaction Information

The execution layer uploads the transaction data, and Meeda will generate a KZG polynomial commitment `Commitment` based on the transaction data. This value can not only uniquely identify the block transaction data, but also be used to subsequently verify the correctness of the data availability certificate. The promise value occupies 96 bytes and is expressed in the `bytes32[4]` format in the contract. The application layer uploads the commitment value and metadata information of the transaction data to blockchains such as Ethereum for persistent storage of the execution layer transaction data.

&nbsp;
Transaction data will be divided into multiple slices, and erasure coding or multiple backup technologies are used for data redundancy, which greatly increases the cost of node evil. Memolabs' RAFI technology is used to realize risk-aware fault identification, this increases overall data availability in less time.

### Download Data

Any node in the execution layer can easily access transaction data based on the commitment value of the transaction data. With Meeda's data availability guarantee, light nodes will be able to verify the availability of data without downloading all data.

### Generate Verification Information

According to the kzg polynomial commitment scheme and decentralization features, a pseudo-random number `rnd` will be generated regularly by the contract to challenge the storage layer. The storage layer determines which files need to be sampled and challenged based on the random number, and then generates data availability certificates for these files to be challenged based on the random number.

&nbsp;
The data availability layer will ensure that the pseudo-random number is unpredictable and will only be generated within a specific short period of time, thereby preventing the storage layer from using time loopholes to falsify and avoid punishment.

&nbsp;
The storage layer needs to submit a certificate within a certain short period of time. If the certificate is not submitted after the validity period, it will be regarded as a challenge failure. Specific nodes in the storage layer will be punished and the data repair mechanism will also be triggered.

&nbsp;
Based on pseudo-random numbers, the sampled files to be challenged are determined to ensure that the selected files are random and unpredictable. The sampling number is calculated based on probability theory to ensure that each challenge has a 99.999% probability of finding faulty data.

### Generate Proof and Submit

The storage layer obtains the data holding certificates of the files to be verified off-chain, aggregates these certificates, and submits the aggregated certificates and aggregated commitment values to the chain.

&nbsp;
Proofs submitted by the storage layer are considered correct by default, thereby reducing verification overhead and reducing data availability costs.

### Multiple Rounds of Interactive Optimistic Verification

After the storage layer submits the proof, in order to reduce the cost of the proof, multiple rounds of interactive optimistic verification are adopted. Meeda assumes that the aggregated commitment value is correct, and any participant can challenge the aggregated commitment value for fraudulent behavior.

&nbsp;
Participants can question the correctness of the aggregation commitment. Since the aggregation proof has been verified to be correct relative to the aggregation commitment, therefore, as long as the aggregate commitment value is correct relative to the pseudo-random number `rnd`, then the data availability certificate will pass.

&nbsp;

For the verification of aggregated commitment values, we adopt multiple rounds of interactive verification. When the amount of data is large, in order to ensure high availability of data, the amount of data for sampling inspection is also correspondingly large. In order to reduce the cost of verification calculation, we divide the aggregated commitment value into ten equal parts. For each challenge, the challenger selects one of them. To challenge the challenge, the data availability layer only needs to perform ten aggregation domain calculations. If the questioned part is proved to be correct by the contract calculation of the data availability layer, then the challenger fails and will forfeit the deposit.

&nbsp;

If through contract calculation, it is found that there is indeed wrong information in the aggregated part being questioned, then the questioner will again divide the questioned part into ten equal parts, select one part for questioning, until the individual block transaction data is finally determined, the questioner and the data The availability layer has reached a consensus on the faulty block transaction data, doubters will be rewarded, and the storage layer node that caused the data failure will be punished.

<img src="..\..\images/onestepproof.png" title="" alt="" data-align="center">

## z

Meeda provides an efficient and simple DA solution method for Ethereum Layer 2, especially Rollups, through kzg polynomial commitment technology, elliptic curve calculation, data availability sampling, erasure code multiple backup redundancy mechanism, efficient and low-cost availability guarantee on the chain, etc.
