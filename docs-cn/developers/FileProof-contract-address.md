# FileProof Contract Address

合约地址：

| FileProofProxy       |   0xBC2BB92F589f332F07f7aE06d0f5080870CE0467  |
| -------------------- | --- |
| **FileProofControl** |   0x6C3F60480e7246EaE194f3F6512E49b392B76D7c  |
| **FileProof**        |   0x7bf3C03fda919e4CfbB1FD79E815FAa51755720d  |

注意：

合约目前部署在Memo-dev链上。

合约代码位于gitlab（目前暂未公开）的`DID-Solidity/tree/memoda/contracts`，经`abigen`转化的合约go代码位于`DID-Solidity/tree/memoda/go-contracts`。

用户通过`FileProofProxy`合约地址，构造`FileProofProxy`合约实例，直接调用`FileProofProxy`合约实例，从而与`FileProof`合约交互。
