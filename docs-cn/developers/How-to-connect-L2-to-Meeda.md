# L2如何接入Meeda

Meeda作为适用于以太坊Layer2的数据可用性解决方案，本文将详细讲解L2如何接入Meeda。

L2链本质上和L1链没有区别，一样是由虚拟机执行交易后，将结果打包到区块中共识上链。之所以执行的更快，gas开销更低主要是因为节点更少更精了，可以通过调整gas计算参数来降低gasfee。因为DA的接入不会干扰链本身的运行，所以L1和L2链都不需要做任何变动。

L2接到L1需要部署一系列的合约，用于完成消息的传递，这也是L2的核心。这些合约在部署后就有固定的地址和相应的事件用于分辨出哪些是与L2L1交互相关的交易。由于DA层的接入只需要修改传递的消息内容，而不影响传递的流程和方式，所以这一块也不需要变动。

L2锚定到L1链，以及验证相关的流程需要L2与L1之间有一层负责交互的角色。这些角色主要分为两个，一个是`batcher`，另一个是`node`。前者主要将L2的交易数据打包存放在到L1链上，后者主要是读取L1链上记录的数据，从而获得可在L2上执行的交易内容。显然这两块都与DA层有关，或者说在L2方案上，这两个角色本身就负责与DA相关的内容，也大多独立出了相关的接口方法。DA的接入就需要变动这两块的相关流程。

剩余的一些挑战角色、监控和提案者和L2的锚定以及安全性有关，具体参考op，这里不再详述。

因此，Rollup接入Meeda，只需要改动`batcher`以及`node`模块的逻辑，应用到Meeda的`submit`和`get`方法即可。

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
