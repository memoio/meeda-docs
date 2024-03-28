# How to Connect L2 to Meeda

## Overview of L2

Here we take the Optimistic Rollup solution in Layer2 as an example to briefly summarize L2:

1. The OP chain is used as the execution layer, and all transactions are executed on this chain. Because the OP chain nodes are of higher quality (fewer and more refined), the synchronization and execution speed will be much faster compared to L1 chains such as Ethereum. In essence, the principle and structure of the chain are not much different from L1.
2. However, because L2 has fewer nodes, its security or decentralization is far less than that of the L1 chain. In order to ensure the security of transactions on the L2 chain, the transaction execution results on the L2 chain need to be anchored to the L1 chain to obtain the same security and credibility as the L1 chain. And this execution result is the state root after the transaction is executed.
3. Although the L2 state root anchored to the L1 chain can be considered to have been confirmed and can no longer be changed, it is still unknown whether the L2 root is a valid and honest state root, and malicious nodes can upload information that is beneficial to themselves. or a tampered state root. In order to ensure the validity of the root, L2-OP adopts the fault-proof method to punish malicious nodes and reward challenging nodes by submitting proofs. This includes optimistic assumptions (there will be nodes for verification during the challenge cycle), sampling verification (2D-RScode to improve verification efficiency) and other details that will not be repeated.
4. The final key issue is **data availability**. If the transaction data on L2 is withheld by the node, even if there is a fault-proof verification mechanism, there is no original transaction data to verify and challenge fraud. In order to ensure that the data is available, the OP's approach is to package a large block of L2 transactions, and finally submit an L1 transaction to the L1 chain. At this time, the L2 transaction data is stored in the `calldata` field of the L1 transaction. This not only ensures that the data copy is on the L1 chain, and any node can obtain the data to verify L2 without the problem of data retention; it also avoids the execution overhead of the L1 chain, because these `calldata` only upload the data chain and does not participate in execution. The actual execution of the transactions corresponding to these data is on the L2 chain.

&nbsp;

At this point, the OP solution, as an expansion solution for Layer 2, can safely and reliably expand Layer 1, improve throughput and reduce transaction fee overhead. But there is still a problem: it cannot reduce data synchronization overhead and storage overhead. The original transaction data still needs to be submitted to the L1 chain, and the full nodes on L1 still need to synchronize these transaction data. Even if there are some other solutions (such as ERC-4337) to compress some transaction data, the effect is still limited. The most intuitive manifestation is that these transaction data still require high gas fees to be stored on the L1 chain, although these fees are byte fees rather than execution fees.

&nbsp;

The fundamental reason is that Layer2-OP regards the Ethereum L1 chain as its own DA layer (Data Availability). Meeda stores data off-chain, and stores the index used to obtain the data and the proof of commitment to ensure data reliability on-chain. While ensuring data availability, it also reduces synchronization overhead and storage overhead on the chain, maximizing the expansion of the blockchain. And because Meeda is an independent layer, external access only needs to use its `get` and `submit` interfaces, and ensuring data security relies on the corresponding challenge and verification mechanism of the Meeda layer, so the access to Meeda is also very Simple and efficient, compatible with any Layer2 chain.

## Connect to Meeda

The following will briefly introduce the principle of access, and use the OP as an example to introduce Meeda access.

&nbsp;

Prerequisite: There is a functioning Layer2, and Layer1 is acting as the DA layer, storing data from Layer2 through `calldata` or `blob`.

&nbsp;

Quoting DA's client package: `https://github.com/memoio/go-da-client`

&nbsp;

Access method:

1. After introducing the package into the project, you can create a module to initialize the client, call the interface, and specify the Meeda identification code.
2. In the module responsible for reading the `calldata` or `blob` data of L1 and restoring it into transaction data that can be executed by L2 (such as `op-node` in op-stack), add the module built in step 1. When the Meeda identification code is read, the client goes to the Meeda layer to obtain the original transaction data, and the subsequent process remains unchanged.
3. In the module responsible for packaging L2 transaction data into the L1 layer (such as `op-batcher` in op-stack), add the module built in step 1. After packaging the L2 transaction data and preparing to encapsulate it into the `calldata` of the L1 transaction, upload this part of the transaction data to the Meeda layer through the client, and use the positioning index (URI, such as CID, MID, commitment, etc.) returned after the upload is successful. The ID of the location data resource) is encapsulated into the `calldata` of the L1 transaction as a replacement, and the subsequent process remains unchanged.

&nbsp;

The impact of access on the original process is very small, because if an error occurs during the use of Meeda, it will automatically fallback back to the original process, and continue according to the original process. There is no need to adjust the parameters of the function or exit the process early. It is a noninvasive, nonaggressive implant.

## Example

Take [optimism](https://github.com/memoio/optimism) as an example:

### Step 1

After introducing the package, create the module directory using Meeda client: `op-memo`.

The internal `da_client.go` encapsulates the interface for initializing the client and calling the DA layer.

The internal `da.go` specifies the Meeda identifier in `calldata` indicating the Meeda layer index.

The internal `cli.go` specifies the command line parameters: the RPC address to access the Meeda service.

### Step 2

The execution logic for reading `calldata` data in optimization and restoring it into transaction data that can be executed by L2 is in `DataFromEVMTransactions` of `op-node/rollup/derive/calldata_source.go`:

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

When the Meeda identification code is read in the `calldata` of the transaction sent to `BatchInboxAddress` (`case memo.DerivationVersionMemo:`), the client goes to the Meeda layer to obtain the original transaction data, and the subsequent process remains unchanged.

You need to initialize a Meeda client for `op-node` at startup:

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

The specific initialization process is completed when `op-node` starts and will not be described in detail here.

### Step 3

In optimization, the execution logic responsible for packaging L2 transaction data to the L1 layer is in `sendTransaction` of `op-batcher/batcher/driver.go`. But here we keep the original processing flow unchanged, and only adjust it when using calldata to generate transactions, that is `calldataTxCandidate`:

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

After obtaining the transaction data to be encapsulated, first try to upload the data to the Meeda layer through DAClient. If there is no problem with the upload, the returned positioning index together with Meeda's identification code will be encapsulated and replaced with the original transaction data. The subsequent process remains unchanged and this part of the transaction data will be encapsulated into the L1 transaction to be packaged. If the upload fails, the original process is maintained and the complete transaction data is encapsulated into the L1 transaction to be packaged.

Similarly, you need to initialize a Meeda client for `op-batcher` at startup:

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

The specific initialization process is completed when `op-batcher` is started and will not be detailed here.
