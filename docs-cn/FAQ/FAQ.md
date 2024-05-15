# FAQ

1.What's Data Availability(DA)?

DA不仅仅包含数据的存储，还需要保证任意节点都能够访问数据。以太坊扩容方案Layer2 Rollup执行交易、提交交易之后，验证节点需要对交易数据进行验证，验证通过的交易才会被Layer1最终确定。验证过程的前提就是能够访问交易数据，因此，交易数据可以被任意节点访问，从而便于验证交易正确性的过程就叫做数据可用性，更多详情请参考：https://ethereum.org/en/developers/docs/data-availability/.

2.What's Memolabs?

[Memolabs](https://memolabs.org/)是一家专注于研究和扩展去中心化云存储技术应用的实验室。我们的使命是为Web3.0构建可靠的基础设施，并开发强大的去中心化存储和Web3应用工具。

3.What's Mefs?

mefs是[Memolabs](https://memolabs.org/)开发的一个低成本、可靠安全的分散式云存储系统，利用了边缘存储设备、以及区块链的不可篡改性和去中心化特性。具体信息可参考[Memolabs](https://memolabs.org/)。

4.What's Meeda?

Meeda是Memolabs为以太坊Layer2，尤其是Optimistic类的Rollup推出的一种数据可用性解决方案，具有低成本、安全可靠的特点。

5.How to combine Ethereum Layer2 with Meeda?

具体请参考`Node/Op Node & Meeda`和`Developers/Connect L2 to Meeda`部分的内容。
