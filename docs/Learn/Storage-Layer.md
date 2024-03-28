# Storage layer

The Meeda architecture consists of a storage layer and a data availability layer. The storage layer is responsible for storing transaction data, and the DA layer is responsible for ensuring data availability. The two modules work together to form the data availability layer (DA Layer) at the blockchain extension level.

## Overview

Meeda's storage layer adopts the Mefs system and configures flexible solutions with erasure coding and multiple backups, thereby greatly improving data availability. Mefs is a decentralized cloud storage system developed by Memolabs. It uses the decentralized characteristics and traceability of blockchain to store data on servers in multiple geographical locations and supports personal idle storage devices at the edge to improve data reliability and availability, and reduce data storage costs. The system is able to provide a high degree of data security and support large-scale data storage and access needs.

&nbsp;

Considering user preferences, Meeda's storage layer is also compatible with storage systems such as IPFS, allowing users to choose flexibly.
