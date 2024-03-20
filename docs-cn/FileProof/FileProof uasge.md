# FileProof使用说明

合约地址：

| FileProofProxy       |                                            |
| -------------------- | ------------------------------------------ |
| **FileProofControl** |                                            |
| **FileProof**        |                                            |
| **Instance**         | 0x10790c26fB42AaDB87c10b91a25090AF1Ff5352E |

使用说明：

合约代码位于`DID-Solidity/tree/memoda/contracts`，经`abigen`转化的合约go代码位于`DID-Solidity/tree/memoda/go-contracts`。

用户通过`FileProofProxy`合约地址，构造`FileProofProxy`合约实例，直接调用`FileProofProxy`合约实例，从而与合约交互。
