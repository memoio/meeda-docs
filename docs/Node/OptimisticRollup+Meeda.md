# Optimistic Rollup and Meeda

Here we will briefly introduce the operation method of Rollup connecting to Meeda as the data availability layer.

&nbsp;

The L2 chain is essentially the same as the L1 chain. After the virtual machine executes the transaction, the results are packaged into blocks and uploaded to the chain by consensus. The reason why the execution is faster and the gas overhead is lower is mainly because there are fewer and more refined nodes. Gasfee can be reduced by adjusting the gas calculation parameters. Because the access of DA will not interfere with the operation of the chain itself, neither the L1 nor L2 chains need to make any changes.

&nbsp;

When L2 receives L1, it needs to deploy a series of contracts to complete the message delivery, which is also the core of L2. After deployment, these contracts have fixed addresses and corresponding events that are used to identify transactions related to L2L1 interactions. Since the access of the DA layer only needs to modify the content of the delivered message, but does not affect the delivery process and method, there is no need to change this part.

&nbsp;

The anchoring of L2 to the L1 chain and the verification-related processes require a layer of interactive roles between L2 and L1. These roles are mainly divided into two, one is `batcher` and the other is `node`. The former mainly packages and stores L2 transaction data on the L1 chain, while the latter mainly reads the data recorded on the L1 chain to obtain transaction content that can be executed on the L2. Obviously these two parts are related to the DA layer, or in the L2 solution, these two roles themselves are responsible for DA-related content, and most of them have independently developed relevant interface methods. DA access requires changes to the related processes of these two parts.

&nbsp;

The remaining challenge roles, monitoring and proposers are related to L2 anchoring and security. Please refer to the op for details and will not be detailed here.

Therefore, to connect Rollup to Meeda, you only need to change the logic of the `batcher` and `node` modules and apply it to the `submit` and `get` methods of Meeda.

For specific operation methods, please refer to `Developers/How-to-connect-L2-to-Meeda`.
