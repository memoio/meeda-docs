# Meeda Detailed Description

This article explains in detail the overall interaction process of Meeda from a developer's perspective for developers to discuss and communicate with.

## Upload Transaction Data

Rollup uploads the transaction data, and Meeda will generate a KZG polynomial `Commitment` based on the transaction data. This value can not only uniquely identify the block transaction data, but also be used to subsequently verify the correctness of the data availability proof. The promise value occupies 96 bytes and is expressed in the `bytes32[4]` format in the contract. Meeda will upload the commitment value and metadata information of the transaction data to blockchains such as Ethereum for persistent storage of the execution layer transaction data.

Transaction data will be divided into multiple slices, and erasure coding or multiple backup technologies are used for data redundancy, which greatly increases the cost of node evil. Memolabs' RAFI technology is used to realize risk-aware fault identification, this increases overall data availability in less time.

## Download Data

Any node in the execution layer can easily access transaction data based on the commitment value of the transaction data. With Meeda's data availability guarantee, light nodes will be able to verify the availability of data without downloading all data.

## Generate Verification Information

According to the kzg polynomial commitment scheme and decentralization characteristics, a pseudo-random number `rnd` will be generated regularly by the contract to challenge the storage node. `rnd` will determine the sample to be sampled. The storage node generates a data availability certificate for the sampled files.

Meeda's verification contract will ensure that the pseudo-random number cannot be predicted and will only be generated within a specific short period of time, thereby preventing storage nodes from using time loopholes to falsify and avoid punishment.

The storage node needs to submit the proof within a certain short period of time. If the proof is not submitted after the validity period, it will be regarded as a challenge failure and will be punished, and data repair will also be triggered.

Based on pseudo-random numbers, the sampled files to be challenged are determined to ensure that the selected files are random and unpredictable.

Meeda's sampling verification is an adaptive verification method. It performs full challenges on new blocks (e.g. blocks in the last 10 epochs) and sample challenges on old blocks. DA needs to ensure 100% that transaction data is available. Therefore, for transaction data that has not been finalized, Meeda will conduct a full challenge to 100% guarantee the availability of the data. For older data, Meeda uses a sampling challenge. We assume that the data failure rate is 1/1000, which means that only 1 in 1000 files is faulty (for storage, this level of evil is not profitable. Assuming 100GB of data, the evil node only stores 99.9GB data, so for him, the risk is greater than the benefit, so the failure rate of 1/1000 is almost the lowest failure rate), then the probability of not finding the wrong data after challenging m times is (1-1/1000) ^m, then the probability of finding fault data is 1-(1-1/1000)^m. We assume that the fault discovery rate is 99.99%, which means that the fault can be found with 99.99% certainty. Then we can get m to be approximately equal to 9205.7. We assume that each sampling challenge is 10,000 data blocks, so that the probability of discovery is greater than 99.99%. Failure data. This adaptive sampling check enables us to 100% guarantee the data availability of recently submitted transactions, and guarantee the data availability of historical transactions with a probability of greater than 99.99%.

## Generate Proof and Submit

The storage nodes generate the data holding proofs of the files to be verified off-chain, aggregates these proofs, and submits the aggregated proofs and aggregated commitment values to the chain.

Meeda's verification contract assumes that the proof is correct. When the proof is submitted to the verification contract, the contract will not be verified immediately. When the light node initiates a challenge, the verification contract will verify the proof, thereby reducing verification overhead and cost of data availability.

## Multiple Rounds of Interactive Optimistic Verification

After the storage node submits the proof, in order to reduce the cost of the proof, multiple rounds of interactive optimistic verification are adopted. Light nodes verify the aggregate commitment value and aggregate proof value off-chain, and if an error is found, the proof can be challenged.

If it is a challenge to the aggregated commitment value, Meeda will adopt multiple rounds of interactive verification. In order to reduce the cost of verification calculations, we divide the aggregate commitment value into ten equal parts. For each challenge, the challenger selects one of them to challenge. The contract only needs to perform the aggregation domain calculation ten times. If the challenged part is proved to be Correct, if the questioner fails, the deposit will be forfeited; if through contract calculation, it is found that there is indeed wrong information in the aggregate part being questioned, then the questioner needs to divide the questioned part into ten equal parts again and select one to initiate the challenge. Until the non-aggregated error commitment value is finally determined, both the challenger and the verification contract reach a consensus on the faulty data, the challenger will be rewarded, and the storage node that caused the data failure will be punished.

<img src="../../images/onestepproof.png" title="" alt="" data-align="center">

## Summarize

Meeda ensures the availability of data through kzg polynomial commitment technology, elliptic curve calculation, data availability sampling, erasure coding multiple backup redundancy mechanism, on-chain verification and other technologies, providing a safe, reliable, and low-cost DA solution for Ethereum Layer 2, especially Optimistic Rollups.
