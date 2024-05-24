# Deploy Meeda Light Node

## Install

This document will guide you how to install Meeda light node.

### Install from source code

Installation of Meeda light node includes the following steps:

1. Delete all the local dependency libraries include meeda-node, and download them again.

   ```bash
   cd $HOME
   ### delete all the local dependency libraries include Meeda-node
   rm -rf meeda-node did-solidity go-did memov2-contractsv2 go-mefs-v2 relay memo-go-contracts-v2
   ### download dependency libraries
   git clone http://132.232.87.203:8088/did/did-solidity.git
   git clone http://132.232.87.203:8088/did/go-did.git
   git clone http://132.232.87.203:8088/508dev/memov2-contractsv2.git
   git clone http://132.232.87.203:8088/508dev/go-mefs-v2.git
   git clone http://132.232.87.203:8088/508dev/relay.git
   git clone http://132.232.87.203:8088/508dev/memo-go-contracts-v2.git
   ### git checkout to the newest branch(go-mefs-v2)
   cd go-mefs-v2
   git checkout 2.7.4
   cd ..
   ## git checkout to the newest branch(did-solidity)
   cd did-solidity
   git checkout meeda-v1.2.0
   cd ..
   ## git checkout to the newest branch(go-did)
   cd go-did
   git checkout meeda-v1.1.0
   cd ..
   ### download meeda-node
   git clone http://132.232.87.203:8088/middleware/meeda-node.git
   ### 
   cd meeda-node
   ### git checkout to the newest branch
   git checkout 1.2.0
   ```

2. Compile meeda-node

   ```bash
   CGO_ENABLED=1 make build
   ```

3. Install meeda-node

   ```bash
   make install
   ```

4. Check if the meeda-node has been installed successfully

   ```bash
   meeda version
   ```

   This command will print the version information, git commit information and compilation date of meeda-node.

You have installed meeda-node, and then you need to deploy meeda-node to access the data availability network of Meeda.

## Deploy

Meeda light node will verify the KZG proofs submitted by Meeda Storage nodes, if the proofs are wrong, then light node will launch multiple rounds of challenges.

General users can access and connect to data availability network of Meeda by Meeda light node. Meeda light node contains the following features:

- Upload and download data.
- Synchronize information of uploaded data, includes data commitment, size and expiration.
- Synchronize KZG aggregation certificate information.
- Verify KZG aggregation certificate information, if the certificcate is found to be incorrect, multiple rounds of challenges will be initiated.

General users deploy light node and then can gain profits through challenge.

### Run Light Node

You need to set `sk` parameter when run Meeda light node. `sk` is used to sign the transactions to Ethereum.

After running Meeda light node, its rpc port will be bounded to 8082.

> **run on product chain**
>
> ```bash
> meeda light run --sk=<private key>
> ```

>**run on dev chain**
>
>```bash
>meeda light run --chain=dev --sk=<private key>
>```

### Close Meeda Light Node

To quit light node normally, please use `Control + C` command on the running terminal.
