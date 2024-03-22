# Meeda是什么

Memolabs推出了一种高效简洁的以太坊DA解决方案，为Layer2 Rollup等执行层提供了可靠的数据可用性保障。根据不同的成本选择，提供不同程度的可用性保障。

这一方案被命名为Meeda，抽取MEMO、Ethereum、data availability中的字母，并且与Memolabs推出的比特币数据可用性解决方案Mooda进行联动，前两个字母共同组成MEMO（Memolabs推出的底层分散式云存储系统）。

## DA问题

对于区块链来说，数据可用性（DA）是一件很重要的事情。众所周知，共识是组成区块链的重要一环，而节点要想达成共识，就必须要保障可靠的数据可用性。节点可以访问到当前区块的交易数据，从而验证交易是否正确，以此防范恶意交易上链，维持区块链正确的“记账”记录，保障区块链的安全可靠性。
但是在像以太坊这样的整体区块链，DA通常作为单个系统设计的一部分，在区块空间有限而区块空间利用率较高时，Gas费会越来越高，单笔交易成本越来越高，用户体验会越差，Web3的发展也会受制。
以太坊最近几年也意识到了扩容问题，开始探索各种链下扩容解决方案，当前，Rollups已经成为流行的解决方案，但是在Web3生态繁荣、区块空间需求较高时，仍然面临Gas费过高问题。

## 当前DA解决方案

基于这种现状，当前主要有两种解决方案。第一种是优化Layer2的存储开销方案，比如以太坊的Danksharding提案 ，目前是采用了EIP-4844，实现了blob交易方式，降低Layer2在依赖以太坊作为数据可用性层的存储开销；第二种是模块化区块链方案，将执行层、共识层、数据可用性层分开设计，比如Celestia项目，旨在通过提供可插拔的共识网络来简化区块链部署。

<img src="..\..\images/now-resolve-method.png" title="" alt="" data-align="center">

## Meeda解决方案

### Meeda概览

当前Layer2作为以太坊的扩展方案，已经能够安全可靠的对Layer1进行扩展，提升吞吐量并降低交易费开销。但其仍存在一个问题：无法降低数据的同步开销和存储开销。原始的交易数据仍然需要提交到L1链上，L1上的全节点仍需要同步这些交易数据。即使有一些别的方案，如ERC-4337来压缩部分交易数据，以及EIP-4844引入了`blob`交易类型，但效果仍旧有限。最直观的体现就是，这些交易数据存放在L1链上仍需要支付高昂的gas费用，尽管这些费用是字节费用而非执行费用。

究其根本，还是因为Layer2将以太坊L1链当作自己的DA层（Data Availability，数据可用性）。而Meeda将数据放在链下存储，获取数据用的索引和保证数据可靠性的承诺证明放在链上存储。在保证了数据可用性的同时，也降低了链上的同步开销和存储开销，最大程度扩展区块链。而保证数据安全性则依靠Meeda层对应的挑战与验证机制，因此Meeda的接入也非常简洁高效，可以兼容任何的Layer2链。

在链下，发布交易的节点将交易信息上传至Meeda，Meeda保障交易信息的可用性，节点可以快速方便地读取交易数据。在链上，Meeda需要定期往链上提交数据可用性证明，证明通过则可以认为用户数据的可用性得到保障，证明失败则会触发数据修复。其中，文件可用性证明的验证方式高效且低成本，采用乐观式多轮交互验证方式、KZG多项式承诺等技术实现这一特点；Meeda对交易数据进行可用性抽查，并且配置纠删码和多备份的灵活方案，从而极大地提高了数据可用性。

### Meeda架构

Meeda分为链下和链上两部分。
链下部分负责存储交易数据，提供数据读取功能，响应数据可用性证明挑战，进行数据修复。
链上部分负责保障数据可用性。主要表现在记录交易数据的承诺值以及元数据信息，生成存储挑战信息，记录链下提交的数据可用性证明，响应可用性欺诈证明，激励数据修复。<img src="..\..\images/da-structure.png" title="" alt="" data-align="center">

## Meeda详细描述

### 上传文件

执行层上传交易数据，Meeda会根据交易数据生成KZG多项式承诺`Commitment`，该值不仅可以唯一识别区块交易数据，也用于后续验证数据可用性证明的正确性。承诺值占用96字节，在合约中用`bytes32[4]`格式表示。应用层将交易数据的承诺值和元数据信息上传至以太坊等区块链中进行执行层交易数据的持久化保存。
交易数据会被分割成多个切片，通过纠删码或多备份技术进行数据冗余，使得节点作恶成本大大增加，运用上 Memolabs的RAFI技术，实现风险感知故障识别，从而以更短的时间提高整体数据的可用性。

### 下载文件

执行层任意节点根据交易数据的承诺值可以轻松访问交易数据，在Meeda的数据可用性保障性下，轻节点将无需下载所有数据即可验证数据的可用性。

### 生成验证信息

根据kzg多项式承诺方案以及去中心化特性，将由合约定期生成一个伪随机数`rnd`，用来对存储层发起挑战。存储层根据该随机数确定哪些文件需要被抽样挑战，之后，再根据该随机数对这些待挑战文件生成数据可用性证明。
数据可用性层将保证该伪随机数无法预测，只会在特定的短时期内生成，从而避免存储层利用时间漏洞造假从而躲避惩罚。
存储层需要在确定的短时期内提交证明，超过了有效时期还未提交证明，则视为挑战失败，存储层特定节点将受到惩罚，并且还会触发数据修复机制。
根据伪随机数，确定抽样的待挑战文件，保证挑选的文件具有随机性、不可预测性。抽样数量根据概率论计算，从而保证每次挑战有99.999%的概率发现故障数据。

### 生成证明并提交

存储层在链下获得待验证文件的数据持有证明，聚合这些证明，将聚合证明和聚合承诺值提交到链上。
存储层提交的证明默认情况下被视为是正确的，从而减小验证开销、降低数据可用性的成本。

### 多轮交互式乐观验证

存储层提交证明后，为了减少证明的代价，采用了多轮交互式的乐观验证方式。Meeda假设聚合承诺值是正确的，任何参与者都可以针对该聚合承诺值进行欺诈行为挑战。
参与者可以质疑聚合承诺的正确性，由于之前已经验证过聚合证明相对聚合承诺是正确的，因此接下来只要聚合承诺值相对于伪随机数`rnd`是正确的，那么该次数据可用性证明就通过。
针对聚合承诺值的验证，我们采取多轮交互式验证。当数据量很大时，为了保证数据的高可用性，抽样检查的数据量也相应很大，为了降低验证计算的成本，我们将聚合承诺值等分成十份，每次挑战，质疑者选中其中一份进行挑战，数据可用层只需要进行十次聚合域计算，若被质疑的部分经过数据可用层的合约计算证明是正确的，那么质疑者失败，会被罚没押金；若通过合约计算，发现被质疑的聚合部分确实是存在错误信息，那么质疑者再次把质疑的部分十等份，选中一份进行质疑，直至最终确定到单独的区块交易数据，质疑者和数据可用层都对该有故障的区块交易数据达成了共识，质疑者将获得奖励，导致数据故障的存储层节点将接受惩罚。

<img src="..\..\images/onestepproof.png" title="" alt="" data-align="center">

## 总结

Meeda通过kzg多项式承诺技术、椭圆曲线计算、数据可用性抽样、纠删码多备份冗余机制、链上高效低成本的可用性保障等技术，为以太坊Layer2，尤其是Rollups ，提供了一种高效简洁的DA解决方案。