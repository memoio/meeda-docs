# FileProof Contract Address

contract address：

| Contract             |               Address                         |
| -------------------- | --- |
| FileProofProxy       |   0xBC2BB92F589f332F07f7aE06d0f5080870CE0467  |
| FileProofControl     |   0x6C3F60480e7246EaE194f3F6512E49b392B76D7c  |
| FileProof            |   0x7bf3C03fda919e4CfbB1FD79E815FAa51755720d  |

Notice:

The contract is currently deployed on the Memo-dev chain.

The contract code is located in `DID-Solidity/tree/memoda/contracts` in gitlab (not yet public), and the contract go code converted by `abigen` is located in `DID-Solidity/tree/memoda/go-contracts`.

The user constructs a `FileProofProxy` contract instance through the `FileProofProxy` contract address and directly calls the `FileProofProxy` contract instance to interact with the `FileProof` contract.
