# L2如何接入Meeda

## L2概述

这里以Layer2中的Optimistic Rollup方案为例，简单概括L2：

1. 将OP链作为执行层，所有的交易都在该链上执行。因为OP链节点更优质（少而精），因此同步与执行的速度相比于L1链如以太坊会快得多。而本质上链的原理和结构与L1没有太大区别。
2. 但因为L2的节点少，因此其安全性或者说去中心化程度远远不及L1链。为了保证L2链上的交易安全，需要将L2链上的交易执行结果锚定到L1链上，以获取与L1链相同的安全性与可信度。而这个执行结果就是执行完交易后的状态根。
3. 虽然通过锚定到L1链上的L2状态根可以认为该L2状态已经被确认，不再可更改，但该L2根是否为有效且诚实的状态根仍不可知，恶意节点可以上传对自己有利的或篡改过的状态根。为了保证根的有效性，L2-OP采用fault-proof的方式，通过提交证明的方式惩罚恶意节点，奖励挑战节点。其中包括乐观假设（挑战周期内有节点做验证）、抽样验证（2D-RScode来提升验证效率）等其他细节不再赘述。
4. 最后的关键问题在于**数据可用性**。如果在L2上的交易数据被节点扣留，那即使有fault-proof的验证机制，也缺乏原始的交易数据用于验证和挑战欺诈行为。为了保证数据可得，OP的做法是将一大块的L2交易打包，最后提交一笔L1交易到L1链上，L2交易的数据此时都存放在L1交易的`calldata`字段内。这样既保证了数据副本在L1链上，任何节点都能获取到这些数据对L2做验证，而不存在数据扣留的问题；也避免了L1链的执行开销，因为这些`calldata`只是把数据上链，而不参与执行，这些数据对应的交易的真正执行是在L2链上。

至此，OP方案作为Layer2的扩展方案，已经能够安全可靠的对Layer1进行扩展，提升吞吐量并降低交易费开销。但其仍存在一个问题：无法降低数据的同步开销和存储开销。原始的交易数据仍然需要提交到L1链上，L1上的全节点仍需要同步这些交易数据。即使有一些别的方案（如ERC-4337）来压缩部分交易数据，但效果仍旧有限。最直观的体现就是，这些交易数据存放在L1链上仍需要支付高昂的gas费用，尽管这些费用是字节费用而非执行费用。

究其根本，还是因为Layer2-OP将以太坊L1链当作自己的DA层（Data Availability，数据可用性）。而Meeda将数据放在链下存储，获取数据用的索引和保证数据可靠性的承诺证明放在链上存储。在保证了数据可用性的同时，也降低了链上的同步开销和存储开销，最大程度扩展区块链。且由于Meeda是独立出来的一层，外部接入只需要使用其`get`和`submit`的接口，而保证数据安全性则依靠Meeda层对应的挑战与验证机制，因此Meeda的接入也非常简洁高效，可以兼容任何的Layer2链。

## 接入Meeda

下面将简单介绍接入的原理，并以OP为例子介绍Meeda的接入。

前提：有正常运作的Layer2，且Layer1正作为DA层，通过`calldata`或`blob`存放来自Layer2的数据。

引用DA的client包：`https://github.com/memoio/go-da-client`

接入方式：

1. 在项目中引入包后，可以创建一个模块用于初始化client、调用接口，并规定Meeda的识别码。
2. 在负责读取L1的`calldata`数据并恢复成可被L2执行的交易数据的模块中（如op-stack中的`op-node`），增加在步骤1中构建的模块。在读取到Meeda的识别码时通过client去Meeda层获取到原本的交易数据，后续流程不变。
3. 在负责将L2的交易数据打包至L1层的模块中（如op-stack中的`op-batcher`），增加在步骤1中构建的模块。在打包完L2交易数据准备封装至L1交易的`calldata`前，通过client将这部分交易数据上传至Meeda层，并将上传成功后返回的定位索引（URI，如CID、MID、commitment等用于定位数据资源的ID符）作为替换封装至L1交易的`calldata`中，后续流程不变。

接入对原流程的影响非常小，因为如果在使用Meeda的过程中出现错误，会自动走fallback回到原本的流程中，按原来的流程继续走下去，不需要调整函数的参数和提前退出流程，是noninvasive、nonaggressive的植入。

## 示例

以[optimism](https://github.com/memoio/optimism)为例：

### 步骤1

引入包后，创建使用Meeda client的模块目录：`op-memo`

内部的`da_client.go`封装了初始化client和调用DA层的接口。

内部的`da.go`规定了在`calldata`中表明是Meeda层索引的Meeda识别码。

内部的`cli.go`则指定了命令行参数：接入Memo DA服务的RPC地址。

### 步骤2

在optimism中读取`calldata`数据并恢复成可被L2执行的交易数据的执行逻辑在`op-node/rollup/derive/calldata_source.go`的`DataFromEVMTransactions`中：

``` golang
// ...

// DataFromEVMTransactions filters all of the transactions and returns the calldata from transactions
// that are sent to the batch inbox address from the batch sender address.
// This will return an empty array if no valid transactions are found.
func DataFromEVMTransactions(dsCfg DataSourceConfig, batcherAddr common.Address, txs types.Transactions, log log.Logger) []eth.Data {
	out := []eth.Data{}
	for _, tx := range txs {
		if isValidBatchTx(tx, dsCfg.l1Signer, dsCfg.batchInboxAddress, batcherAddr) {
			// Meeda: if the calldata is represented in MemoDerivation marker, then fetch it from Meeda layer
			data := tx.Data()
			switch len(data) {
			case 0:
				out = append(out, data)
			default:
				switch data[0] {
				case memo.DerivationVersionMemo:
					log.Info("Meeda: blob request", "id", hex.EncodeToString(tx.Data()))
					ctx, cancel := context.WithTimeout(context.Background(), daClient.GetTimeout)
					blobs, err := daClient.Client.Get(ctx, [][]byte{data[1:]})
					cancel()
					if err != nil {
						log.Warn("Meeda: failed to resolve frame", "err", err)
						log.Info("Meeda: using eth fallback")
						out = append(out, data)
					}
					if len(blobs) != 1 {
						log.Warn("Meeda: unexpected length for blobs", "expected", 1, "got", len(blobs))
						if len(blobs) == 0 {
							log.Warn("Meeda: skipping empty blobs")
							continue
						}
					}
					out = append(out, blobs[0])
				default:
					out = append(out, data)
					log.Info("Meeda: using eth fallback")
				}
			}
		}
	}
	return out
}
```

在发往`BatchInboxAddress`的交易的`calldata`中读取到Meeda的识别码时（`case memo.DerivationVersionMemo:`）通过client去Meeda层获取到原本的交易数据，后续流程不变。

需要在启动时先为`op-node`初始化一个Meeda的client：

```golang
var daClient *memo.DAClient

func SetDAClient(c *memo.DAClient) error {
	if daClient != nil {
		return errors.New("da client already configured")
	}
	daClient = c
	return nil
}
```

具体的初始化流程在`op-node`启动时完成，这里不再细述。

### 步骤3

在optimism中负责将L2的交易数据打包至L1层的执行逻辑在`op-batcher/batcher/driver.go`的`sendTransaction`中。但这里保持原有的处理流程不变，只在用calldata生成交易时调整一下，即`calldataTxCandidate`：

```golang
// ...

func (l *BatchSubmitter) sendTransaction(ctx context.Context, txdata txData, queue *txmgr.Queue[txID], receiptsCh chan txmgr.TxReceipt[txID]) error {
	// ...
}

// ...

func (l *BatchSubmitter) calldataTxCandidate(data []byte) *txmgr.TxCandidate {
	l.Log.Info("building Calldata transaction candidate", "size", len(data))

	// Meeda: try to submit the data to Meeda layer
	ctx, cancel := context.WithTimeout(context.Background(), 30*time.Duration(l.RollupConfig.BlockTime)*time.Second)
	ids, err := l.DAClient.Client.Submit(ctx, [][]byte{data})
	cancel()
	if err == nil && len(ids) == 1 {
		l.Log.Info("Meeda: blob successfully submitted", "id", hex.EncodeToString(ids[0]))
		data = append([]byte{memo.DerivationVersionMemo}, ids[0]...)
	} else {
		l.Log.Info("Meeda: blob submission failed; falling back to eth", "err", err)
	}

	return &txmgr.TxCandidate{
		To:     &l.RollupConfig.BatchInboxAddress,
		TxData: data,
	}
}
```

在获得待封装的交易数据后，先尝试通过DAClient将数据上传至Meeda层。如果上传没有问题，将返回的定位索引连同Meeda的识别码一起封装并替代原本的交易数据，后续流程不变，将这部分交易数据封装到待打包的L1交易中。如果上传失败，则维持原本的流程，将完整的交易数据封装到待打包的L1交易中。

同样地，需要在启动时先为`op-batcher`初始化一个Meeda的client：

```golang
// DriverSetup is the collection of input/output interfaces and configuration that the driver operates on.
type DriverSetup struct {
	Log              log.Logger
	Metr             metrics.Metricer
	RollupConfig     *rollup.Config
	Config           BatcherConfig
	Txmgr            txmgr.TxManager
	L1Client         L1Client
	EndpointProvider dial.L2EndpointProvider
	ChannelConfig    ChannelConfig
	PlasmaDA         *plasma.DAClient
	DAClient         *memo.DAClient
}
```

具体的初始化流程在`op-batcher`启动时完成，这里不再细述。
