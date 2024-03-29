# Data Availability layer

The Meeda architecture consists of a storage layer and a data availability layer. The storage layer is responsible for storing transaction data, and the DA layer is responsible for ensuring data availability. The two modules work together to form the data availability layer (DA Layer) at the blockchain extension level.

## Overview

We know that "Don't trust, verify" is a common motto in the Web3 community. When applied to data availability issues, this means that any node can obtain the transaction data contained in a block submitted by a peer node and independently verify that the blockchain state information it receives is correct by executing all transactions in that block, thus ensuring that state changes proposed by peers exactly match changes independently calculated by the nodes themselves. This means that nodes do not have to trust that the sender of the block is honest. Data loss scenario is also impossible.

Therefore, Meeda needs to design a solution so that block data can be proven to be accessible without trust. In addition to trustlessness and security, cost has always been an important consideration for Web3 users. Therefore, Meeda hopes that the cost of availability proof will be as low as possible and the efficiency will be as high as possible.

Taking into account the trustless characteristics, Meeda uses smart contracts to verify data availability. Smart contracts run on a blockchain that provides high security, thus ensuring the safety and reliability of the verification method in an open and transparent manner.

In order to reduce the overhead of data availability proof and improve the efficiency of data availability proof, Meeda got inspiration from Optimistic Rollup in Layer 2 and implemented optimistic verification and multi-round interactive fraud proof scheme suitable for file availability proof. And the use of KZG polynomial commitment technology makes the verification method efficient and low-cost. This provides a simple, efficient, low-cost, scalable, and decentralized DA solution.

Meeda's data availability layer (DA layer) mainly includes functions to ensure data reliability and data accessibility, specifically: generating proof information, verifying data availability proof, storing metadata information, and incentive mechanisms. See `Developers` for more details.
