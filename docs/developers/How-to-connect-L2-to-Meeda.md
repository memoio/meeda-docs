# How to Connect L2 to Meeda

Meeda is a data availability solution suitable for Ethereum Layer 2. This article will explain in detail how L2 connects to Meeda.

The L2 chain is essentially the same as the L1 chain. After the virtual machine executes the transaction, the results are packaged into blocks and uploaded to the chain by consensus. The reason why the execution is faster and the gas overhead is lower is mainly because there are fewer and more refined nodes. Gasfee can be reduced by adjusting the gas calculation parameters. Because the access of DA will not interfere with the operation of the chain itself, neither the L1 nor L2 chains need to make any changes.

When L2 connects to L1, it needs to deploy a series of contracts to complete the message delivery, which is also the core of L2. After deployment, these contracts have fixed addresses and corresponding events that are used to identify transactions related to interactions of L1 and L2. Since the access of the DA layer only needs to modify the content of the delivered message, but does not affect the delivery process and method, there is no need to change this part.

The anchoring of L2 to the L1 chain and the verification-related processes require a layer of interactive roles between L2 and L1. These roles are mainly divided into two, one is `batcher` and the other is `node`. The former mainly packages and stores L2 transaction data on the L1 chain, while the latter mainly reads the data recorded on the L1 chain to obtain transaction content that can be executed on the L2. Obviously these two parts are related to the DA layer, or in the L2 solution, these two roles themselves are responsible for DA-related content, and most of them have independently developed relevant interface methods. DA access requires changes to the related processes of these two parts.

The remaining challenge roles, monitoring and proposers are related to L2 anchoring and security. Please refer to the op for details and will not be detailed here.

Therefore, to connect Rollup to Meeda, you only need to change the logic of the `batcher` and `node` modules and apply it to the `submit` and `get` methods of Meeda.

## Connect to Meeda

The following will briefly introduce the principle of access, and use the OP as an example to introduce Meeda access.

Prerequisite: There is a functioning Layer2, and Layer1 is acting as the DA layer, storing data from Layer2 through `calldata` or `blob`.

Quoting DA's client package: `https://github.com/memoio/go-da-client`

Access method:

1. After introducing the package into the project, you can create a module to initialize the client, call the interface, and specify the Meeda identification code.
2. In the module responsible for reading the `calldata` or `blob` data of L1 and restoring it into transaction data that can be executed by L2 (such as `op-node` in op-stack), add the module built in step 1. When the Meeda identification code is read, the client goes to the Meeda layer to obtain the original transaction data, and the subsequent process remains unchanged.
3. In the module responsible for packaging L2 transaction data into the L1 layer (such as `op-batcher` in op-stack), add the module built in step 1. After packaging the L2 transaction data and preparing to encapsulate it into the `calldata` of the L1 transaction, upload this part of the transaction data to the Meeda layer through the client, and use the positioning index (URI, such as CID, MID, commitment, etc.) returned after the upload is successful. The ID of the location data resource) is encapsulated into the `calldata` of the L1 transaction as a replacement, and the subsequent process remains unchanged.

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

When the Meeda identification code(`case memo.DerivationVersionMemo:`) is read in the `calldata` of the transaction sent to `BatchInboxAddress`, the client goes to the Meeda layer to obtain the original transaction data, and the subsequent process remains unchanged.

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
