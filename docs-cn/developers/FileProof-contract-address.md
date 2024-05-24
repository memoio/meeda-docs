# FileProof Contract Address

合约地址：

* chain: memo product

| Contract        | Address |
|----------------------|--------------------------------------------|
| FileProof        | 0x58C3Ab98546879a859EDBa3252A9d38E43C9cbee     |
| ControlFileProof | 0x6eEc7578dBAD9dcc1CA159A9Df0A73233548b89a     |
| ProxyFileProof   | 0x0c7B5A9Ce5e33B4fa1BcFaF9e8722B1c1c23243B     |
| Instance         | 0xbd16029A7126C91ED42E9157dc7BADD2B3a81189     |

* chain: memo dev

| Contract        | Address |
|----------------------|--------------------------------------------|
| FileProof        | 0x7bf3C03fda919e4CfbB1FD79E815FAa51755720d     |
| ControlFileProof | 0x6C3F60480e7246EaE194f3F6512E49b392B76D7c     |
| ProxyFileProof   | 0xBC2BB92F589f332F07f7aE06d0f5080870CE0467     |
| Instance         | 0x10790c26fB42AaDB87c10b91a25090AF1Ff5352E     |

合约代码位于gitlab（目前暂未公开）的`DID-Solidity/tree/memoda/contracts`，经`abigen`转化的合约go代码位于`DID-Solidity/tree/memoda/go-contracts`。

用户通过`FileProofProxy`合约地址，构造`FileProofProxy`合约实例，直接调用`FileProofProxy`合约实例，从而与`FileProof`合约交互。
