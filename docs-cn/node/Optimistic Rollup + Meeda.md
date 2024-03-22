# Optimistic Rollup 与 Meeda

Ethereum Layer2中的Optimistic Rollup方案简单概括：

1. 将OP链作为执行层，所有的交易都在该链上执行。因为OP链节点更优质（少而精），因此同步与执行的速度相比于L1链如以太坊会快得多。而本质上链的原理和结构与L1没有太大区别。
2. 但因为L2的节点少，因此其安全性或者说去中心化程度远远不及L1链。为了保证L2链上的交易安全，需要将L2链上的交易执行结果锚定到L1链上，以获取与L1链相同的安全性与可信度。而这个执行结果就是执行完交易后的状态根。
3. 虽然通过锚定到L1链上的L2状态根可以认为该L2状态已经被确认，不再可更改，但该L2根是否为有效且诚实的状态根仍不可知，恶意节点可以上传对自己有利的或篡改过的状态根。为了保证根的有效性，L2-OP采用fault-proof的方式，通过提交证明的方式惩罚恶意节点，奖励挑战节点。其中包括乐观假设（挑战周期内有节点做验证）、抽样验证（2D-RScode来提升验证效率）等其他细节不再赘述。
4. 最后的关键问题在于**数据可得性**。如果在L2上的交易数据被节点扣留，那即使有fault-proof的验证机制，也缺乏原始的交易数据用于验证和挑战欺诈行为。为了保证数据可得，OP的做法是将一大块的L2交易打包，最后提交一笔L1交易到L1链上，L2交易的数据此时都存放在L1交易的`calldata`字段内。这样既保证了数据副本在L1链上，任何节点都能获取到这些数据对L2做验证，而不存在数据扣留的问题；也避免了L1链的执行开销，因为这些`calldata`只是把数据上链，而不参与执行，这些数据对应的交易的真正执行是在L2链上。

至此，OP方案作为Layer2的扩展方案，已经能够安全可靠的对Layer1进行扩展，提升吞吐量并降低交易费开销。但其仍存在一个问题：无法降低数据的同步开销和存储开销。原始的交易数据仍然需要提交到L1链上，L1上的全节点仍需要同步这些交易数据。即使有一些别的方案（如ERC-4337）来压缩部分交易数据，但效果仍旧有限。最直观的体现就是，这些交易数据存放在L1链上仍需要支付高昂的gas费用，尽管这些费用是字节费用而非执行费用。

究其根本，还是因为Layer2-OP将以太坊L1链当作自己的DA层（Data Availability，数据可得性）。而Meeda将数据放在链下存储，获取数据用的索引和保证数据可靠性的承诺证明放在链上存储。在保证了数据可得性的同时，也降低了链上的同步开销和存储开销，最大程度扩展区块链。且由于Meeda是独立出来的一层，外部接入只需要使用其`get`和`submit`的接口，而保证数据安全性则依靠Meeda层对应的挑战与验证机制，因此Meeda的接入也非常简洁高效，可以兼容任何的Layer2链。

下面将简单介绍接入的原理，并以OP为例子介绍Meeda的接入。

## 原理

L2链本质上和L1链没有区别，一样是由虚拟机执行交易后，将结果打包到区块中共识上链。之所以执行的更快，gas开销更低主要是因为节点更少更精了，可以通过调整gas计算参数来降低gasfee。因为DA的接入不会干扰链本身的运行，所以L1和L2链都不需要做任何变动。

L2接到L1需要部署一系列的合约，用于完成消息的传递，这也是L2的核心。这些合约在部署后就有固定的地址和相应的事件用于分辨出哪些是与L2L1交互相关的交易。由于DA层的接入只需要修改传递的消息内容，而不影响传递的流程和方式，所以这一块也不需要变动。

L2锚定到L1链，以及验证相关的流程需要L2与L1之间有一层负责交互的角色。这些角色主要分为两个，一个是`batcher`，另一个是`node`。前者主要将L2的交易数据打包存放在到L1链上，后者主要是读取L1链上记录的数据，从而获得可在L2上执行的交易内容。显然这两块都与DA层有关，或者说在L2方案上，这两个角色本身就负责与DA相关的内容，也大多独立出了相关的接口方法。DA的接入就需要变动这两块的相关流程。

剩余的一些挑战角色、监控和提案者和L2的锚定以及安全性有关，具体参考op，这里不再详述。

## OP接入

在`optimism`项目的源码中，有两部分与DA层相关：

1. `op-batcher/batcher/driver.go`的`sendTransaction`，将交易的数据放在`txmgr`的候补交易中。交易的数据就是L2层打包的一帧（Frame）数据。此时为了接入DA层，可以通过Meeda层的`submit`接口将交易的数据先上传到Memo，获得回执中的ID索引，随后用该ID索引替换原来的交易数据。
2. `op-node/rollup/derive/calldata_source.go`的`DataFromEVMTransactions`，用于取L1链上发送给`batch inbox address`的交易，并取出其`calldata`字段用于后续的执行使用。接入DA后，首先分析其交易数据的前缀（1字节），如果符合DA的格式，则将后面的数据认作为ID索引，并通过该索引到Meeda上通过`get`接口取回对应的原始交易数据。

这些植入对原流程的改动很小，因此易于兼容和使用。

为了调用Meeda上的`submit`和`get`接口，需要初始化一个Meeda的DABackend client来访问DA的RPC。在`op-memo`中抽象并封装了这两个接口，并提供相应的参数用于初始化client。

在OP相关角色的启动中也加入Meeda的启动配置，具体为`op-node`以及`op-batcher`，需要在其启动的时候为其初始化一个Meeda的client。

最后在部署流程中设置参数`OP_BATCHER_DA_RPC`和`OP_NODE_DA_RPC`指向Meeda运行的HTTP RPC即可。