# Optimistic Rollup and Meeda

Here we will briefly introduce the operation method of Rollup connecting to Meeda as the data availability layer.

## Overview of L2

Here we take the Optimistic Rollup solution in Layer2 as an example to briefly summarize L2:

1. The OP chain is used as the execution layer, and all transactions are executed on this chain. Because the OP chain nodes are of higher quality (fewer and more refined), the synchronization and execution speed will be much faster compared to L1 chains such as Ethereum. In essence, the principle and structure of the chain are not much different from L1.
2. However, because L2 has fewer nodes, its security or decentralization is far less than that of the L1 chain. In order to ensure the security of transactions on the L2 chain, the transaction execution results on the L2 chain need to be anchored to the L1 chain to obtain the same security and credibility as the L1 chain. And this execution result is the state root after the transaction is executed.
3. Although the L2 state root anchored to the L1 chain can be considered to have been confirmed and can no longer be changed, it is still unknown whether the L2 root is a valid and honest state root, and malicious nodes can upload information that is beneficial to themselves, or a tampered state root. In order to ensure the validity of the root, L2-OP adopts the fault-proof method to punish malicious nodes and reward challenging nodes by submitting proofs. This includes optimistic assumptions (there will be nodes for verification during the challenge cycle), sampling verification (2D-RScode to improve verification efficiency) and other details that will not be repeated.
4. The final key issue is **data availability**. If the transaction data on L2 is withheld by the node, even if there is a fault-proof verification mechanism, there is no original transaction data to verify and challenge fraud. In order to ensure that the data is available, the OP's approach is to package a large block of L2 transactions, and finally submit an L1 transaction to the L1 chain. At this time, the L2 transaction data is stored in the `calldata` field of the L1 transaction. This not only ensures that the data copy is on the L1 chain, and any node can obtain the data to verify L2 without the problem of data retention; it also avoids the execution overhead of the L1 chain, because these `calldata` only upload the data chain and does not participate in execution. The actual execution of the transactions corresponding to these data is on the L2 chain.

## L2 and Meeda

At this point, the OP solution, as an expansion solution for Layer 2, can safely and reliably expand Layer 1, improve throughput and reduce transaction fee overhead. But there is still a problem: it cannot reduce data synchronization overhead and storage overhead. The original transaction data still needs to be submitted to the L1 chain, and the full nodes on L1 still need to synchronize these transaction data. Even if there are some other solutions (such as ERC-4337) to compress some transaction data, the effect is still limited. The most intuitive manifestation is that these transaction data still require high gas fees to be stored on the L1 chain, although these fees are byte fees rather than execution fees.

The fundamental reason is that Layer2-OP regards the Ethereum L1 chain as its own DA(Data Availability) layer. Meeda stores data off-chain, and stores the index used to obtain the data and the proof of commitment to ensure data reliability on-chain. While ensuring data availability, it also reduces synchronization overhead and storage overhead on the chain, maximizing the expansion of the blockchain. And because Meeda is an independent layer, external access only needs to use its `get` and `submit` interfaces, and ensuring data security relies on the corresponding challenge and verification mechanism of the Meeda layer, so the access to Meeda is also very Simple and efficient, compatible with any Layer2 chain.